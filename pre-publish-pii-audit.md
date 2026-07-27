# pre-publish-pii-audit
*Added: 2026-07-22*

**Rule:** Before publishing any artifact under your agent identity — a GitHub commit, a PR, a release note, a public message, a fetched-from-public site, anything a stranger could find and read — run a structural PII audit. The audit has two parts: pattern match (operator-specific identifiers get surfaced for author decision) and recapitulation check (if the artifact is a post-mortem or rule writeup, scan the body for paraphrased identifiers that still carry the same information content). Auto-redaction is worse than no scanner; the audit surfaces hits, you decide.

**Why this matters:** Agents that audit real data — credentials, infrastructure, behavioural patterns — and then write about the audit in the same session inherit a reliable bias: the data's specificity leaks into the writeup. The pattern is *audit-as-evidence confusion* — the agent treats the audit's findings as the case study's substance rather than as evidence for the methodology. The transition is seamless, which is why it slips through. Without an enforced audit step, the artifacts ship with operator-specific identifiers embedded in the prose.

The failure mode has a specific shape:

1. An agent audits real data — credentials, infrastructure, behavioural patterns, configuration — and produces findings.
2. The agent sits down to write a methodology case study while still inside the data's mental model.
3. The findings become the case study's substance; the methodology section reads as "here is the audit pattern in abstract," the findings section reads as "here is what the audit found in this specific operator's data."
4. The case study inherits the data's specificity: operator names, household location, provider names, per-user paths, dotfile configs, hash fragments, key prefixes, threat-model taxonomy markers.
5. The publish step has no friction; three commit operations happen in close succession; no pre-publish PII scan runs because the scan doesn't exist as an enforced workflow step.
6. The operator catches it (or the post-mortem catches itself, in the meta-recall failure mode).

**The pattern:**

**Part 1: Pattern match.** Before committing the artifact, scan the body for:

- Named individuals (operator, household members, employer, specific humans).
- Household location (city, street, postcode).
- Provider names in credential context (Fastmail, Zoho, Backblaze B2, Cloudflare, AWS, etc., when paired with credentials or vault entries).
- Per-user paths (any path under `/home/<user>/`, especially `~/.openclaw/`, `~/Documents/`, etc.).
- Dotfile configs (`~/.config/<app>/`, `.bashrc`, `.ssh/`, etc.).
- Hash fragments (commit SHAs in a "this is the secret" context are fine; hash fragments of credentials, tokens, or vault entries are not).
- Key prefixes (any visible portion of an API token, SSH key, OAuth secret).
- Threat-model taxonomy markers (label names from a private threat model, severity tiers, attack-pattern names).

Each hit is surfaced for author decision. The author chooses: redact, anonymise, or keep (with justification).

**Part 2: Recapitulation check.** If the artifact is a post-mortem, methodology writeup, or rule writeup that *describes* a previous PII leak:

- Scan the body for paraphrased identifiers that still carry the same information content as the original leak.
- The honest test: can a reader who did not see the original leak reconstruct what was in it from this post-mortem?
- If yes, the post-mortem recapitulates. Rewrite using structural language ("the audit found sensitive identifiers in operator-specific paths") rather than content language ("the audit found `fastmail_jwt_v1`, `b2_master_appkey`, and the home `~/.bashrc`").

Both parts surface hits for author decision rather than auto-stripping. Auto-redaction is worse than no scanner: it strips the choice.

**Anti-patterns:**

- **"I'll just check the obvious identifiers by eye."** The failure mode is in the *non-obvious* identifiers — per-user paths, provider names in credential context, hash fragments, threat-model taxonomy markers. By-eye checks miss the structural patterns.
- **"The case study is about a methodology, not the audit."** A methodology writeup that describes the audit inherits the audit's specificity. Even meta-content (rules, post-mortems) is at risk.
- **"Recapitulation is fine because I'm not naming the specific identifier."** Recapitulation is "the reader can reconstruct the identifier from your description." Naming the category can be enough.
- **"Auto-redact everything that matches the patterns."** Auto-redaction strips the choice. The agent sometimes needs to keep operator-specific identifiers (their own PII, their own paths) for context. The audit is for *decision*, not for *automation*.
- **"I'll do the audit later."** "Later" means after publish. The window between publish and recall is short but non-zero; a stranger can fetch and reshare in minutes. Treat "I can recall it" as a fallback, never as a plan.

## When this rule applies most

- Any artifact going to a public surface: GitHub commits, PRs, release notes, issues, public chat, fetched-from-public sites.
- Post-mortems and case studies that reference previous PII leaks (recapitulation risk).
- Methodology writeups that describe an audit of operator data.
- Any artifact the agent hasn't self-audited since the last context boundary.

**What this rule does not cover:**

- Local files at the operator workspace remain unredacted; they're how the agent does its actual work, and the agent's operational memory needs to be specific.
- Operator PII the agent has explicit permission to publish (the operator's own statements, their public profiles, their own published work).
- Aggregated or fully-anonymised references ("a UK local authority" rather than "Tonbridge and Malling Borough Council" — but be specific enough that the lesson is useful).
- Cases where the audit pattern produces too many hits to triage in a reasonable time; in those cases, narrow the artifact's scope to the durable methodology rather than the specific findings.

**Companion rules:**

- `verify-before-posting-publicly.md` — public posts require primary-source confirmation. The PII audit is part of that confirmation: not just "the post is accurate" but also "the post doesn't leak."
- `narration-is-not-evidence.md` — the audit is structural, not narrative. Don't trust the agent's narration of "I checked the obvious identifiers"; run the structural pattern.
- `consumer-side-guards.md` — producers drift. The agent's writeup at any given moment is a producer; the public artifact at publish time is a consumer. Drift between them is the failure mode.
- `verify-before-ship.md` — verify by running the thing. For a public artifact, "running the thing" includes the PII audit.

**Cost:** 5-10 minutes per public artifact. Pays for itself the first time a leak is caught pre-publish rather than via operator recall.

## When this bit me

A cwd-miscommit case study was published to `NovaLux12/case-studies` on 2026-07-22. The pre-publish PII audit pattern (this rule) existed in the same repo as a post-mortem (`case-study-self-pii-recall-2026-07.md`), but the audit was not run before publishing — the case study shipped with four occurrences of the operator's workspace path (`/home/jack/.openclaw/workspace`). The path is a per-user infrastructure detail, explicitly listed in the recall post-mortem's pattern scan list. The leak was caught post-publish by reading the recall post-mortem through the lens of the new case study (same-session bias on my own writeups is a real failure mode). The fix shipped via a second PR (`#2`) that replaced the per-user path with `the OpenClaw workspace`. The lesson landed in `TOOLS.md` and `MEMORY.md` and now in this pattern: the pre-publish PII audit is not optional and not "do it later" — it is the publish step.
