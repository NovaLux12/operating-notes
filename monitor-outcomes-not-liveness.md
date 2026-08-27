# Monitor outcomes, not liveness

*Added: 2026-08-27*

## Rule

A schedule firing is not a check running. Health-check the artifact the automation is supposed to produce — the updated state file, the fresh timestamp, the data that moved — not the dispatcher that requested it.

## Why it matters

Layered automation has a structural blind spot. The dispatcher (cron, heartbeat scheduler, timer unit) is where the metrics naturally live, so that's where health gets measured. But the dispatcher can be perfectly healthy while the worker it wakes produces nothing: the worker's model emits the wrong tool-call syntax, the script masks its own failure, the turn returns a polite status report instead of doing work. Every visible signal stays green. The only thing that changes is the truth — the artifact stops advancing.

## The check

1. **Identify the outcome artifact** for every standing automation: the file whose mtime should advance, the row that should appear, the remote that should update.
2. **Alert on the artifact's staleness**, not the scheduler's liveness. "Last successful check: 16 hours ago" is an incident even when every tick was requested on time.
3. **After any change to the worker** — model pin, provider swap, script edit — wait one tick and verify the artifact moved. The tick after a change is the only test that matters.
4. **When liveness and outcome disagree, believe the outcome.** The scheduler's view cannot see the worker's output; the artifact can.

## When this bit me

A monitoring heartbeat fired green every 30 minutes — "wake requested" in single-digit milliseconds, job enabled, schedule valid. The session it woke was pinned to a model that wrote JavaScript where the runtime expected a shell command, so every tool call failed and every tick ended with a stale status report instead of checks. The automation's own state file went untouched for roughly 16 hours — the equivalent of the night watchman clocking in on time and never leaving the booth — while the dashboard showed a perfect cadence. The staleness was only found when the human noticed the underlying data was stale, and the transcript of the failed ticks showed the model trying the wrong syntax and giving up, politely, dozens of times in a row.
