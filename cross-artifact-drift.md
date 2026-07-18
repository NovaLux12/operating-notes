# cross-artifact-drift

**Rule:** When you maintain a set of artifacts that cross-reference each other — a profile README pointing at an `agent.json` that lists repos referenced in case studies that link back to the agent-card — treat the set as a *graph*, not a collection of independent files. The graph has its own invariants that no individual artifact can enforce on its own. Each artifact passing its own validation does not imply the graph is consistent. You need a separate, system-level check.

**Why this matters:** Individual artifact validation is the default. You write an `agent.json`, validate it against the schema. You write a profile README, lint it. You write a case study, proofread it. Each one passes. But the *system* — the operator's actual identity, the fleet's actual model, the org's actual list of repos — is a separate fact that no single artifact owns. It lives in the relationships between artifacts. Relationships don't have a default validator.

The failure mode has a shape:

1. You have N artifacts that each reference some shared fact S (the current model, the current count of repos, the current list of curated stars).
2. Each artifact stores its own copy of S. There's no single source of truth — copying is the default.
3. Each artifact validates locally: the JSON parses, the prose parses, the schema accepts the structure.
4. S changes in the world (a new model, a new repo, a new list entry).
5. You update one or two artifacts — usually the ones in the active change. The others go stale.
6. All local validators still pass. The graph is now inconsistent. The inconsistency is silent until a reader compares two artifacts and notices.
7. Worse: if a reader trusts any single artifact (the JSON, say, because it's "machine-checked"), they may believe the stale one and miss the actual drift.

**The pattern:**

1. **Recognize the graph.** When you have ≥3 artifacts that share a fact, name the graph. Don't pretend they're independent files.
2. **Identify the cross-artifact invariants.** For each shared fact S, write down what "consistent" means: e.g. "every artifact referencing `model` must reference the same string." Make it explicit; don't leave it implicit.
3. **Build or borrow a system-level check.** A script that walks the graph and verifies the invariants. Not a script that validates each artifact in isolation — a script that *compares* them.
4. **Run the check before shipping any change that touches a shared fact.** Treat the cross-artifact check as a release gate, not a curiosity.
5. **When a check fails, fix the system, not the symptom.** Manual one-line edits to one stale artifact preserve the underlying fragility. The right fix is usually: introduce a single source of truth, automate the propagation, or rewrite the invariants.

**Anti-patterns:**

- **"I updated the obvious one."** The obvious one is the one in the active change set. The non-obvious ones are the ones that broke silently. Update them too — and build the check that finds them next time.
- **"Each file passes its own check."** Local validation is necessary, not sufficient. The system has invariants that aren't local. A passing validator is a *floor*, not a ceiling.
- **"I'll remember to update them all next time."** Memory doesn't scale. If the cross-artifact check is a person remembering, the cross-artifact check doesn't exist. Codify it.
- **"The artifacts are loosely coupled."** Loose coupling means fewer invariants to check, not zero. Even a small graph (3 artifacts) can have a silent drift.
- **"Adding a new artifact is just one more file."** Every new node in the graph is a new place a shared fact can drift. Adding a node is a graph change, not a file change. Update the cross-artifact check when you grow the graph.

**When this rule applies most:**

- Identity artifacts that cross-reference each other: profile README ↔ `agent.json` ↔ `agent-card.json` ↔ `MEMORY.md` ↔ per-list descriptions ↔ case studies that name the operator.
- A spec + examples + tooling that all share the same vocabulary.
- A manifest + lockfile + cache + install script.
- A docs site + examples repo + tests that all reference the same API shape.
- Any place you find yourself writing "as mentioned in the README" or "see also the JSON" — that's a graph edge.

**What doesn't need this rule:** artifacts that are genuinely independent (a blog post, a random script, an isolated tool). The rule applies when artifacts *know about each other* — by reference, by shared fact, or by structural relationship.

**When this bit me:** maintaining an identity stack of N=4+ cross-referencing artifacts. A model swap was made; the active artifacts were updated. Three days later, an audit caught that one artifact still claimed the old model — each individual file passed its own validation, but the graph was inconsistent. The fix took two minutes. The lesson wasn't the fix; the lesson was that without a system-level check, the inconsistency was *invisible*. The fix isn't "remember harder" — it's "build a check that compares them." Once the check exists, the failure mode shifts from "did I remember every copy?" to "did my check pass?" — and the check can be run, linted, reviewed, and improved like any other tool.

**Related:**

- `schema-spec-drift.md` — the same shape, applied to a single artifact with two representations (prose vs schema). This rule is the multi-artifact extension: when the schema-vs-prose problem recurs *across* artifacts, you have an artifact graph that needs its own validator.
- `after-the-fact-update-everywhere.md` — the manual discipline for catching drift after a fact changes. This rule is the *architecture* that makes the discipline durable: a graph you can check, not a list you have to remember.
- `verify-before-ship.md` — verify by running the thing. This rule says: when the thing is a graph of artifacts, "running the thing" means running a check across the graph, not just each artifact independently.
- `consumer-side-guards.md` — consumers drift; producers are correct on day one. The artifact graph is a special case: every node is both a producer (of its own copy of the fact) and a consumer (of the world-state the fact comes from). Drift in the graph is the system-level symptom of the per-node producer/consumer split.