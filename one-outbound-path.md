# one-outbound-path

**Rule:** For any irreversible external action (email send, public post, message, signup, payment, deletion), use exactly one shared outbound function. The function renders inputs to the final form, logs the action locally, and is the only path. Ad-hoc scripts get binned. There are no exceptions.

**Why this matters:** When the outbound path forks (script A for plain text, script B for HTML, script C for threaded replies, script D for raw `.md`, the system CLI, an MCP integration, a `curl`), each path has its own bugs. The agent has to remember which path is which. The recipient sees "this was sent by some process" without knowing which one. The audit trail is split across tools. The render output is inconsistent.

The single-path rule enforces a discipline: *every* outbound action goes through the same function. The function does three things and does them all:

1. **Render** — convert the input (a draft, a markdown file, a structured payload) to the final form the recipient will see (plain text, HTML, formatted post). No raw `.md` source reaching a recipient. No unrendered template placeholders.
2. **Log** — write a local copy of the rendered output, plus addressing metadata (recipient, sender, subject, message-ID, sent timestamp, status). The log is the audit trail and the write-ahead log for "did we actually send this?"
3. **Return** — give the agent a structured result so the agent can verify (recipient accepted? message-ID returned? status code 200?).

Then there's a two-gate test for *any* new real action (not test sends):

- **Gate A — Test pass.** Test sends go to a known-safe destination (your own inbox, your own audit account). Cover the relevant shapes: plain text, threaded reply, attachment, HTML, a deliberate bounce to verify the failure path. Pipeline is unproven until proven.
- **Gate B — Consolidate.** Bin every other ad-hoc outbound script. The shared function is the only path. "I'll keep sender.py for emergencies" is not consolidation.

Both gates must be passed before any real send to anyone outside the test pass. Gate A without Gate B means the pipeline works but the next agent will reach for the wrong script. Gate B without Gate A means the pipeline is single but never tested.

This rule applies to more than email. It applies to *any* irreversible external action:

- Email (the canonical case)
- Public posts (Mastodon, GitHub comments, blog)
- Signups and account creations
- Payments
- Subscriptions and recurring charges
- Destructive admin actions (account deletion, repo deletion)

**The pattern:**

1. Decide on the shared function (`scripts/email/send.py`, `scripts/social/post.py`, `scripts/payments/pay.py`). One per *kind* of action.
2. Make it the only path. Bin or `chmod 000` the ad-hoc scripts. Delete them. Move them to `archive/` if you must, but the bin is the goal.
3. The function must render, log, and return — all three.
4. Before any real action, run Gate A on a test destination. Verify the relevant shapes.
5. For real actions, use the function. No exceptions. "It's just one post" is the drift.

**Anti-patterns:**

- "I'll just use curl this once." → no, use the function.
- "The function is too rigid for this case." → extend the function. Don't fork it.
- "This is just a test, no need to log." → log anyway. The test is also a path. The log is what makes "did this actually happen?" answerable.
- "I'll fix the other scripts later." → bin them now. "Later" is "never".
- "I need a different script for the API." → wrap the API in the same function. Don't expose the API as a separate path.

**The "no exceptions" rule matters.** The exception is the future drift. Once one path forks, the next fork is easier.

**When this bit me:** A daily email pipeline used `sender.py` for the original test pass and several other one-off scripts (a CLI mail tool, a PowerShell `Send-MailMessage` wrapper, a raw SMTP Python script) accumulated alongside it. When a multi-vendor takedown campaign started, the agent reached for whichever script was closest at hand. Result: three separate log paths, three different rendering behaviours, two scripts that didn't render markdown at all (raw `.md` source was shipped as the email body — recipients saw literal `#` and `**` characters). The two-gate rule caught it: test pass to the agent's own inbox covered the shapes; `sender.py` and the other ad-hoc scripts were binned; the single `scripts/email/send.py` is now the only path. Every subsequent send has been through the function.

**Related:**

- `verify-before-posting-publicly.md` — Gate A is the verification. Test the pipeline before the real post.
- `wal-not-mental-notes.md` — the log is a write-ahead log for outbound actions.
- `after-the-fact-update-everywhere.md` — if the rendering or addressing changes, every consumer (recipient expectation, log format, audit trail, downstream consumers) updates in lockstep.
- `verify-before-ship.md` — the function's return value is itself an output that should be verified, not just trusted.