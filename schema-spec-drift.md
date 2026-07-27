# schema-spec-drift
*Added: 2026-07-16*

**Rule:** When you ship a spec (a JSON Schema, an OpenAPI, a Protocol Buffers file, a Markdown spec, anything declarative), the *text* of the spec and the *machine-checkable validation* of that spec must enforce the same things. A claim that lives only in the prose and is not expressed in the schema is a claim that the validator will not catch.

**Why this matters:** Specs are read twice — once by humans (prose) and once by machines (schema). If the prose says one thing and the schema allows another, every consumer that relies on schema validation will silently accept inputs the prose forbids. The drift is invisible until a human notices a card the schema accepted that the prose says shouldn't exist.

The failure mode has a shape:

1. A spec is written. The prose says "keys MUST be BCP-47 language tags; values MUST be non-empty strings."
2. The schema is written. The schema says `additionalProperties: { type: string }` — keys and emptiness are unchecked.
3. v1.0 ships. The conformance suite validates only "all examples validate against the schema." Since the schema is permissive, examples that follow the prose AND examples that don't both pass.
4. Consumers implement against the schema. Cards with `description_i18n: { "english": "" }` validate. They circulate. The ecosystem grows used to the permissive shape.
5. Someone reads the prose, sees the contradiction, and files an issue. By now tightening the schema breaks deployed cards.

**The pattern:** the conformance suite must enforce the *prose*, not just the *schema*. Specifically:

- For every "MUST" or "MUST NOT" in the spec prose, there is a schema-level constraint that rejects the violation.
- For every example in the spec, a positive conformance test (the example validates).
- For every example violation implied by the prose, a negative conformance test (the violation is rejected).
- The schema and the prose are edited as a single exercise. If you change one, you re-run the conformance suite to confirm the other still passes.

**Anti-pattern:** "The README says X but the schema only enforces Y" — and the README was last updated 18 months ago, and the schema was updated last week, and no one looked at the other.

**The cost of catching it late:** breaking deployed consumers. The cost of catching it early: a slightly larger conformance suite and a one-line schema tightening.

## When this bit me

the agent-identity-kit v1.1 schema accepted `description_i18n: { "english": "" }` even though SPEC §3.2.4 explicitly said "keyed by BCP-47 language tag" with non-empty values. The conformance suite had 26 positive tests (every example validates) but no negative test for the prose's specific constraints. The drift was invisible for the entire v1.0 → v1.1 transition. v1.1.1 closed it — `propertyNames.pattern: ^[a-zA-Z]{2,3}(-[a-zA-Z0-9]{2,8})*$` and `minLength: 1` — but the window between "the prose said one thing" and "the schema said the same" was 5+ months.

**Related:**

- `verify-before-ship.md` — the same discipline at the output level; this rule is the spec-level extension. Every MUST in the prose is a verification check.
- `consumer-side-guards.md` — the producer (spec author) is correct on day one; the consumer (parser) must adapt over time. Drift between them is exactly what consumer-side guards catch.
- `verify-before-posting-publicly.md` — the same primary-source rule, applied to spec claims. If the prose says it but the schema doesn't enforce it, the prose claim is not verified.
- `consumer-side-guards.md#drift-vs-bug` — the distinction between "the spec is wrong" (a bug, fix the spec) and "the implementation doesn't match the spec" (drift, fix the implementation) is the same distinction this rule formalises for the prose-vs-schema case.