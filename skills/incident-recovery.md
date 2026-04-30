---
name: incident-recovery
description: When a long-running service freezes — pipeline, queue, worker, sync job — the recovery sequence is the same regardless of language or stack. Investigate before killing; verify the kill is data-safe; restart with a verification probe; capture the contention pattern as memory so the next instance is faster. The pattern beats writing a project-specific runbook for every service.
tier: universal
---

# Incident recovery

A service that has frozen — stopped making forward progress while still running — looks alarming. The temptation is to kill it immediately and restart. That's the right *eventual* move, but doing it without diagnosis throws away the only artefact that explains why the freeze happened: the live state of the stuck process and the resources it holds.

This skill is the universal recovery sequence: diagnose, verify-data-safe, kill, restart, verify, capture pattern. The same shape works for a database write-lock contention, a queue worker that stopped consuming, an HTTP client wedged in a TCP retry, a cron job blocked on a missing file lock, a daemon waiting on a socket that will never accept.

## When to apply

A service is stuck if all of these hold:
- The process is alive (in process table, holding its PID).
- It has not made forward progress for longer than its expected work cycle (queue depth not decreasing, no new log lines, last commit/transaction timestamp old).
- Restarting it would resume work that was previously flowing.

Don't apply this skill to:
- Crashed processes (already exited; restart is the only move).
- Misconfigured processes (running fine but doing the wrong thing — that's a config fix, not an incident).
- Slow-but-progressing processes (work is happening; back off and observe).

## The sequence

### Step 1 — Diagnose without touching state

Read-only probes only. The goal is to identify what the stuck process is waiting on.

```bash
# Process is alive?
ps -p <PID> -o pid,etime,stat,cmd

# What is it waiting on? — kernel-level
cat /proc/<PID>/wchan          # current syscall wait channel
cat /proc/<PID>/status | head  # state, threads, resource usage

# What files / sockets / locks does it hold?
lsof -p <PID>

# What dependencies might be the contention point?
# (db lock, file lock, socket connection, child process, network endpoint)
```

For database-backed services, check the lock state directly:

```bash
# SQLite WAL state
sqlite3 <db> "PRAGMA wal_checkpoint(PASSIVE);"
# Output: 0|<wal_pages>|<checkpointed> — ideally 0|0|0

# PostgreSQL — locks held + waited on
psql -c "SELECT * FROM pg_stat_activity WHERE state != 'idle';"
psql -c "SELECT * FROM pg_locks WHERE NOT granted;"
```

For queue workers, check the queue:

```bash
# Queue depth not decreasing? Worker present in queue's "active" set?
redis-cli LLEN <queue>
redis-cli LRANGE <active-set> 0 5
```

The probe output answers a single question: **what is the process waiting on, and is the thing it's waiting on capable of unblocking it?** If yes, wait. If no, the process is genuinely stuck.

### Step 2 — Verify the kill is data-safe

The cheapest mistake here is killing a process mid-write. Before sending any signal:

- **Database transactions in progress?** For SQLite, `wal_checkpoint(PASSIVE)` returning `0|0|0` confirms the WAL is empty — no in-flight writer state. For PostgreSQL, look at `pg_stat_activity` for `xact_start` on the stuck process; if non-null, it has an open transaction that will roll back on kill (acceptable; explicit data loss on commits-in-flight, not acceptable for some).
- **File writes mid-flight?** Check `lsof -p <PID>` for files opened with write mode; cross-reference against the work being done.
- **Network requests in flight?** Less critical for kill safety; the receiving end will see a connection drop and handle it (or not).
- **Held locks?** A process holding a file lock or DB lock will release on kill. Confirm no other process is *waiting* on a lock that needs the stuck process to release it cleanly (usually fine; the kill releases via OS).

If the kill is not data-safe, you have a different problem: you need to wait, route around, or escalate. Document the unsafe condition in the post-mortem.

### Step 3 — Send the signal, escalate if needed

```bash
# Clean shutdown first — gives the process a chance to flush
kill -TERM <PID>

# Wait for clean exit (typically 5-30 seconds)
sleep 30
ps -p <PID> > /dev/null && echo "still alive" || echo "exited cleanly"

# Force-kill only if SIGTERM didn't take
kill -KILL <PID>
```

Always SIGTERM before SIGKILL. SIGKILL is the right move for processes that ignore SIGTERM, but it gives no chance for graceful shutdown — you lose the post-mortem opportunity and may leave temporary files or partial state behind.

### Step 4 — Restart with a verification probe

Don't restart and walk away. Restart, then run the probe that originally diagnosed the freeze:

```bash
# Re-launch the service
<your service start command>

# Same probe that diagnosed the freeze, ~30s later
ps -p <new-PID> -o pid,etime,stat
<your service-specific health check>
<queue depth probe — should be decreasing>
```

If the probe shows the same symptom returning within minutes, the kill was the right move but the underlying contention is still present. Move to step 6 — there's a structural problem, not just a transient.

### Step 5 — Capture the pattern as memory

After every freeze recovery, write a memory entry capturing:
- **What was the contention shape?** "Two writers on the same SQLite WAL." "Queue worker waiting on an unreachable HTTP endpoint." "File lock held by a parent process that exited without releasing."
- **What probe identified it?** Make the probe reproducible next time.
- **What was data-safe vs not?** So the next responder doesn't have to re-derive.
- **What permanent fix would prevent recurrence?** Often: a single-writer lock contract, a circuit breaker on the dependency, a timeout on the lock acquisition. The fix may be its own ticket; the memory pre-pays the diagnosis cost.

The memory is more valuable than the recovery because freeze patterns recur. Without the memory, the third recovery costs as much as the first; with it, the third costs ten minutes.

### Step 6 — Structural fix, separate session

If the freeze recurs after a clean recovery, the contention is structural. Common shapes:
- **Multiple writers on a serialised resource.** Move to a single-writer pattern; queue writes; partition the resource.
- **No timeout on a blocking dependency.** Add a timeout; add a circuit breaker; surface the dependency health as a separate metric.
- **Cascading retries that hold the lock during retry.** Release the lock between retries; cap retry budget.
- **Resource leak (file handles, connections, memory).** Fix at the source; restart is not a fix.

The structural fix is its own design exercise. Open a ticket; do not try to ship it during the recovery session.

## Anti-patterns to refuse

- **Kill -9 first.** Throws away post-mortem signal. Always SIGTERM first.
- **Restart loops.** "It's stuck, kill it, it works for an hour, kill it again." Means a structural problem is being papered over. Each recurrence is data; capture the pattern after the second instance and ticket the fix.
- **Skipping the data-safe check under urgency.** Urgency does not change physics; mid-write kills produce corrupt state.
- **Restarting without a verification probe.** "I killed it and started it again" is not a diagnosis; the freeze may immediately resume.
- **Treating each freeze as a unique incident.** Most freezes in a long-running system fall into ~3-5 contention patterns. Once you've recovered each one, you have a runbook.
- **Reaching for destructive commands as recovery shortcuts.** `truncate`, `drop`, `force-push`, `rm -rf` of state directories — these turn an incident into a recovery operation. Investigate first.

## Pairs with

- `diagnose-before-acting` — the diagnose-step here is an instance of that broader pattern.
- `pre-ship-adversary` — when you ship a structural fix in step 6, the adversary asks "does this fix introduce a new contention? Is the new lock contract enforceable?"
- `memory-write` — the contention pattern memory is the load-bearing artefact.
- Your project's first-time-action gate — restarting a critical service is itself a state-mutating action; the gate's discipline applies.
