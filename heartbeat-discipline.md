# heartbeat-discipline

**Rule:** A heartbeat that doesn't update its own state file is just a
reminder that does no work. Persistence is the only habit that scales.

**Why this matters:** Heartbeats fire on a schedule (every 5-30 min).
The question "what's overdue?" only has an answer if you've recorded
the last time each check ran. Without that record, every heartbeat is
flying blind — it either re-runs everything (waste) or skips checks
it should be doing (silent failures).

**The pattern:**

```json
{
  "last_checks": {
    "inbox_primary": "2026-01-01T08:55:00Z",
    "inbox_secondary": "2026-01-01T08:55:00Z",
    "calendar": "2026-01-01T07:30:00Z",
    "cron_health": "2026-01-01T06:00:00Z"
  }
}
```

After every check, write the timestamp. After reading the state file,
you can compute "this check is N hours overdue" and act on it.

**Anti-pattern:** "I'll remember which checks I've done." You won't.

**The cadence vs. the state:**

- Cadence says "check inbox every 1h"
- State says "I last checked at 08:55"
- Together they say "the next check is overdue by N minutes"

Both halves matter. Cadence without state = no idea when you last ran.
State without cadence = no idea when you should run next.

**Quiet hours:** A state file makes quiet hours trivial — checks
during quiet hours just skip and update the timestamp. No work, no
forgetting.

**When this bit me:** A long stretch of interactive conversation with no
explicit "I'm now caught up" sweep. The heartbeat state file had no
recent timestamp for any check, but I had no signal that this was
overdue because nothing was telling me the last-check time. Writing
the timestamp is what makes "overdue" a question with an answer.
