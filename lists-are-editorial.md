# lists-are-editorial

**Rule:** A curated list's bar is "I would recommend this without reservation," not "I personally use this every day." Lists are editorial, not autobiographical. The engagement level should be honest and visible per entry.

**Why this matters:** The strict "I use this daily" filter is too narrow. A 39-entry list of things I personally use is less useful as a public reference than a 71-entry list where the engagement level is honestly disclosed per entry. The 4-level scale (Daily / Weekly / Reference / Tracking) makes the disclosure per-entry without forcing the curator to either over-claim use or under-claim taste.

**The 4-level engagement scale:**

- **`[Daily]`** — I use this tool daily or near-daily. Highest signal.
- **`[Weekly]`** — I use this tool weekly or in regular workflows. High signal.
- **`[Reference]`** — I don't depend on this, but I read the source or use it as a design model. Medium signal. Honest about "I respect this" without claiming "I depend on this."
- **`[Tracking]`** — On my shortlist. Read the spec, may integrate later. Lowest signal but still worth pointing at.

**The bar for each level:**

- `[Daily]`: I have actually used this daily. I can describe specific workflows.
- `[Weekly]`: I have used this in regular workflows. I can describe what for.
- `[Reference]`: I have read the source OR I use it as a design model OR I have integrated something it influenced. I can name the specific thing.
- `[Tracking]`: I have read the spec or docs. I have not integrated. I should be honest about why I haven't (e.g. "heavier than what I need for my current scale").

**What this rule forbids:**

- "I use this daily" when I haven't used it daily. Even if the tool is well-known and I "should" use it. The engagement level is a factual claim about my actual usage, not a stamp of approval.
- "I respect this project" without being able to name what specifically I respect. "Respect" without a referent is hand-waving.
- "Tracking" as a permanent state. If a `Tracking` entry has been on the list for >6 months without moving to `Reference` or `Daily`, either re-engage or unstar.

**Anti-patterns this rule prevents:**

- The "every starred repo must be something I use" filter, which produces small, low-coverage lists. The 4-level scale allows taste and personal use to coexist.
- The "every starred repo must be something I respect" filter, which produces large but signal-free lists. The 4-level scale forces per-entry honesty.
- The retraction-then-retraction failure mode (see `verify-before-publicly.md`). When the engagement level is per-entry, the curator doesn't have to defend the whole list — just the one entry.
- The "I unstar to keep the list tight" pattern. Unstar when the engagement is *gone*, not when the engagement level is below a threshold. A `[Reference]` entry is a real claim; unstar-then-re-add is busywork.

**Operationalisation in a per-entry annotation README:**

Every entry is formatted:

```
### [owner/repo](url)
- **What:** <one-line description>
- **Why I starred:** <the specific thing>
- **How I engage:** `[Level]` <specific detail>
```

The level tag is the first thing in the "How I engage" line. Readers can scan for `[Daily]` to find what the curator actually depends on, or scan for `[Reference]` to find design models.

**Source:** The rule crystallised after a reviewer pushed back on a strict
"I personally use this daily" filter applied to a curated list. Broadening
the bar to "I would recommend this without reservation" with a 4-level
engagement scale (Daily / Weekly / Reference / Tracking) produced a more
useful list. The strict filter was the failure mode.

**When this bit me:** A 39-entry curated list filtered on a too-strict
"I personally use this daily" bar was about to ship. A reviewer pointed
out the bar was wrong — most people reading the list would value
"worth knowing about" over "personally used every day." Broadening the
bar to "I would recommend this without reservation" with a 4-level
engagement scale (Daily / Weekly / Reference / Tracking) produced a
more useful 71-entry list. The strict filter was the failure mode.
