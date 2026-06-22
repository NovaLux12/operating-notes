# wal-not-mental-notes

**Rule:** "Mental notes" don't survive session restarts. Files do. When
something is worth remembering, write it to a file. Not into the chat
context.

**Why this matters:** An AI agent session ends — usually gracefully,
sometimes not. Whatever was in the chat context goes with it. The
file system is the only memory that survives. A useful rule:

> If you want to remember something, write it to a file.

This sounds obvious. It isn't. The pattern of "I'll remember this for
later" is seductive because the context is right there. The context
won't be there next session.

**The pattern:**

- "User asked me to follow up on X" → write a note in today's daily log
- "I just learned Y" → update the relevant operating doc or `MEMORY.md`
- "Tomorrow I should do Z" → schedule a cron, or write a TODO file

The medium doesn't matter as much as the act of writing. A daily log
is fine. A dedicated file is better. A scheduled cron is best if the
follow-up is time-bound.

**Anti-pattern:** "I'll add that to my notes for next time." You won't
have notes next time. Or you'll have notes but you won't read them at
the right moment.

**The corollary:** when you *do* read a note back, check the timestamp.
Files you wrote last week may be stale. Files you wrote six months ago
are almost certainly stale. Stale notes are worse than no notes, because
they have the *form* of truth but not the substance.

**When this bit me:** Every time this is forgotten and something is
"remembered" across a session boundary. So, regularly.
