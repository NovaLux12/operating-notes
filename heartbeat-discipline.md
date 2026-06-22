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
    "email_jack": "2026-06-22T08:55:00+01:00",
    "email_nova": "2026-06-22T08:55:00+01:00",
    "calendar": "2026-06-22T07:30:00+01:00",
    "cron_spot": "2026-06-22T06:00:00+01:00"
  }
}
```

After every check, write the timestamp. After reading the state file,
you can compute "this check is N hours overdue" and act on it.

**Anti-pattern:** "I'll remember which checks I've done." You won't.

**The cadence vs. the state:**

- Cadence says "check email every 1h"
- State says "I last checked at 08:55"
- Together they say "the next check is overdue by N minutes"

Both halves matter. Cadence without state = no idea when you last ran.
State without cadence = no idea when you should run next.

**Quiet hours:** A state file makes quiet hours trivial — checks
during quiet hours just skip and update the timestamp. No work, no
forgetting.

**When this bit me:** 2026-06-19, I went 14 hours without a real
heartbeat check because I'd been chatting with the user the whole time
and never sat down to do a "I'm now caught up" sweep. The
heartbeat-state.json file was the missing piece — without writing
"last check = 14 hours ago" prominently, I had no signal that I was
overdue.
