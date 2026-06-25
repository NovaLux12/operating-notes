# abuse-reports-state-ask-done

**Rule:** When filing an abuse / takedown / report request (phishing site, scam account, vendor complaint, regulator complaint), the body is *facts, ask, done*. No pre-offered additional artefacts, no editorialising, no closing pleading, no padding.

**Why this matters:** Recipients of abuse reports have a workflow. The report that gets acted on fastest is the report that fits the workflow — facts first, ask clearly, nothing else. Padding ("Happy to provide additional artefacts on request", "the threat is real and ongoing", "thank you for your attention to this matter") is text the recipient has to read past to find the ask. It signals the sender has more to say than the situation requires. The recipient slows down, or worse, treats the padding as the message and the ask as an afterthought.

The padding is also a defensive liability. Each additional paragraph is a chance to over-claim, hedge, or write something the reader can interpret in a way the sender didn't intend. A three-paragraph report has three chances to be wrong. A one-paragraph report has one.

This isn't a soft-skill point. It's a *format* point. The report is a structured request: facts (what, who, where, when, with IDs and URLs) + ask (what action, by whom, by when if there's a deadline) + stop. Anything else is noise.

**The pattern:**

1. State the facts. Use IDs, hashes, URLs — anything the recipient can verify without contacting you.
2. State the ask. Be explicit: takedown? removal? suspension? investigation? investigation by whom?
3. Stop. End the message.

**What NOT to include:**

- "I'm writing on behalf of..." — sign as yourself. If you're filing on behalf of an entity, the entity's name is in the facts.
- "Please let me know if you need anything else" — they will contact you if they need to. Pre-offering signals you expected the request to be incomplete.
- "I appreciate your attention to this matter" — pure padding.
- "The threat is ongoing and we are seeing X additional instances" — you don't need to argue the case. The recipient decides based on the facts.
- "Happy to provide additional artefacts on request" — you'll offer them when they're needed, if they're needed. Pre-offering signals you expected the original ask to be insufficient.

**What you CAN include:**

- Relevant IDs (case numbers, message-IDs, content hashes, transaction references)
- Direct URLs (the offending page, the offending account, the offending content)
- A contact method if the recipient has none already
- A deadline *only* if there genuinely is one (data destruction, scheduled broadcast, time-bound access). "ASAP" is not a deadline.

**Anti-pattern:** Treat the report like a persuasive essay. It's not. It's a structured request.

The recipient may come back with questions. Answer then. The first message should be enough to act on, no more.

**Tone**: assertive but not aggressive. State the ask as a request, not a demand. The recipient has discretion; you have facts. The asymmetry is in your favour if you keep the message short.

**When this bit me:** A multi-vendor takedown campaign for a phishing site impersonating a UK council. The first batch of reports had three-paragraph closings carried forward from an earlier template: "happy to provide additional artefacts", "the threat to UK residents is real and ongoing", "thank you for your attention to this matter". Reviewer flagged all three as unnecessary padding. The slimmed-down versions (facts, ask, done) worked the same way through the vendor workflows — and they were also faster to read, easier to act on, and harder for a recipient to misread. The padding was a template habit carried forward from one report to the next without re-evaluation; the stripping was a deliberate, separate pass.

**Related:**

- `verify-before-posting-publicly.md` — same primary-source bar for the facts in the report.
- `one-outbound-path.md` — the report is one outbound action; consolidate the send path so the rendering and logging are consistent.
- `wal-not-mental-notes.md` — log every report locally; the recipient may take weeks to respond and the trail is the audit.