# cron-architecture

**Rule:** When scheduling periodic work, pick the right cron shape for
the job. There are two fundamentally different shapes in agent runtimes,
and mixing them up creates silent failures.

**The two shapes:**

| Shape | Behaviour | Use when |
|---|---|---|
| **systemEvent** to main session | Drops a prompt into the main session | Interactive work that needs main-session attention |
| **isolated agentTurn** | Spawns a sub-agent that does the work and reports back | Background checks, maintenance, "do this now" |

**The trap:** writing a "check if X needs updating" prompt as a
`systemEvent` to main. If main is busy, the prompt just sits there.
The check doesn't happen. No alert fires. Days later, someone
notices.

**The right tool:**

- "Send me a morning briefing" → `systemEvent` to main (interactive)
- "Run a security bulletin scan at 06:00" → `isolated agentTurn`
- "If unread emails > 5, ping me" → `isolated agentTurn` (decides
  whether to ping, doesn't need to be in main)
- "Reflect on yesterday and update MEMORY.md" → `isolated agentTurn`

**The decision tree:**

1. Does the work need conversational context from the main session?
   If yes → `systemEvent`.
2. Does the work need to be visible to the user immediately as it
   happens? If yes → `systemEvent`.
3. Otherwise → `isolated agentTurn`.

Most crons are #3. Most cron bugs are from getting this wrong.

**Sub-isolation:** An isolated cron can spawn its own sub-agents for
fan-out. Use this for "do N parallel things, then summarise" patterns.

**Failure alerts:** Even isolated crons should have a failure-alert
path. If the cron fails three times in a row, ping the user.
Otherwise silent failures pile up.

**When this bit me:** 2026-06-14, mailbox heartbeat was wired as a
`systemEvent` to main. Main was busy with other work. The heartbeat
prompt sat in the queue for 4 hours. By the time main got to it,
the urgent unread count had gone from 3 to 0 (Jack had read them
himself). I missed four real items.
