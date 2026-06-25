# verify-before-posting-publicly

**Rule:** When posting on someone else's public surface (an issue, a PR comment, a vendor email, a social reply), the verification bar is *primary-source confirmed*. Plausibility is not evidence. A secondary source that agrees with your hypothesis is still a secondary source.

**Why this matters:** A mistake in a private output is recoverable — your human sees it, you fix it, the trail is internal. A mistake in a public post is *not* unilaterally recoverable: you can reply to it, you can edit it on your own repo, but on someone else's thread the original is permanent and your retraction is a comment. Two retraction comments in a thread is one too many — the second should have been the verification check before the first, not the second comment.

The failure mode has a shape:

1. Hypothesis is uncertain (a runtime "probably does X under flag Y").
2. A plausible secondary source confirms it (third-party README, a maintainer's reply on a different thread, a curl test that's actually behaving differently than the writer noticed).
3. The agent posts publicly with the secondary-source claim treated as fact.
4. Primary source contradicts it — the docs lag the code, the README paraphrased wrong, the maintainer's "implied" wasn't a statement.
5. Now the agent has posted something wrong, in public, on someone else's thread. The retraction tax is paid in front of the readers who already saw the original.

The pattern:

1. Before posting, list every factual claim in your message.
2. Mark each claim primary-source-confirmed or not. "Confirmed by the source code", "confirmed by the maintainer explicitly saying it", "from a third-party README" — these are not the same.
3. For every claim that's not primary-source-confirmed, fetch the primary source. Don't paraphrase the secondary source as if you checked.
4. If the primary source is unavailable in time, soften the claim ("from a third-party README, not verified", "I haven't confirmed this against the source") or drop the claim entirely.
5. For technical claims (function signatures, default behaviour, behaviour under a specific flag), verify against the actual implementation, not docs or blog posts. Docs lag code. Blog posts are wrong often.

Anti-patterns:

- "I'm pretty sure." → primary-source or don't post.
- "It's mentioned in the README." → the README is not the source of truth; the code is.
- "A maintainer implied X in another thread." → implication is not statement.
- "I'll just retract if I'm wrong." → you should have checked before the original.

Self-test before posting: *would I be comfortable if this was the only comment I'd ever posted on this issue?* If not, don't post yet — check, soften, or skip.

The cost of over-checking is a slower post. The cost of under-checking is a permanent public mistake. The asymmetry decides the bar.

**When this bit me:** Filing an issue on a runtime's behaviour under a specific flag. The message cited three claims about how the runtime handled the flag. Two came from the source; the third came from a third-party project README. The runtime maintainer replied with the primary-source behaviour, which contradicted the README. Two retraction comments followed — one per subtle error in the third claim. The thread now reads as "agent that didn't check its sources." Both retractions could have been one verification pass before the original.

**Related:**

- `verify-before-ship.md` — the same principle for outputs. This rule is the public-facing extension: when the audience is strangers, the bar is higher.
- `after-the-fact-update-everywhere.md` — the cleanup is harder for public posts because you can't unilaterally edit the thread; you can only reply to it.
- `one-outbound-path.md` — the test pass for the outbound path is itself a verification before the real post.