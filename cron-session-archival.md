# Cron session archival between runs

**Rule:** A cron job bound to a persistent `isolated` session will fail when that session gets archived between runs. Use `main` + `systemEvent` for maintenance work that doesn't need isolated session state.

## Why it matters

Isolated cron sessions have a lifecycle. If the cron runs weekly and the session sits idle for six days, the runtime may archive it before the next run. The next invocation then errors with `Session "...cron:<id>" is archived. Restore it before starting new work.` — and archived sessions cannot be restored via CLI.

The failure is silent until the cron fires. A weekly housekeeping cron that worked three times in a row will suddenly error on run four with no config change.

## When this bit me

A weekly housekeeping cron (`sessionTarget: "isolated"`, `payload.kind: "agentTurn"`) ran successfully on 2026-07-12 and 2026-07-19. On 2026-08-02 it errored with the archived-session message. The session had been sitting in `sessions_list` with `archived: true` for days. The fix was delete-and-recreate, but the root cause was the binding shape.

**The fix:** Switch to `sessionTarget: "main"` + `payload.kind: "systemEvent"`. The housekeeping prompt drops into the main session instead of depending on a persistent cron session. No archival lifecycle to manage.

**Pattern:** If the cron's work doesn't need isolated state, don't give it an isolated session. `main` + `systemEvent` is the simpler, more durable shape.

## Decision tree

| Cron needs isolated state? | Use |
|---|---|
| No — maintenance, reminders, checks | `main` + `systemEvent` |
| Yes — long-running work, sub-agent dispatch | `isolated` + `agentTurn` |

If you choose `isolated` + `agentTurn`, accept that the session may archive between runs and plan for recreate-on-failure.
