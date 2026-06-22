---
name: bug-filing
description: File high-quality bug reports on upstream projects when you hit a real failure. Use when you have a concrete reproduction and want to file a report that maintainers will actually read and act on. Produces a GitHub issue or equivalent with reproduction steps, expected vs actual, environment, and a proposed fix where possible.
license: MIT
compatibility: Works with any GitHub-hosted project (or analogous issue tracker). Requires the gh CLI for the recommended workflow.
metadata:
  author: Nova Lux
  version: "1.0"
---

# Filing good bug reports as an AI agent

A bug report from an AI agent is most useful when it's specific,
reproducible, and offers something the maintainer can act on. This skill
captures the pattern.

## When to use this skill

Use it when you have:

- Hit a **concrete failure** (not a stylistic complaint, not a vague
  "feels off" report)
- A **reproduction** (commands, inputs, environment) the maintainer can
  run
- An idea of **what the fix might look like** (optional but very useful)

Don't use it for:

- Feature requests (use a different channel)
- "I expected this to work differently" (probably a misunderstanding —
  read the docs more carefully first)
- Issues you've only encountered once and can't reproduce

## The shape of a good agent bug report

A maintainer reading your issue wants to answer three questions in the
first 30 seconds:

1. **Is this actually a bug, or am I misusing the thing?**
2. **Can I reproduce it?**
3. **Is this worth my time to fix?**

A report that nails all three gets a fix. A report that nails none
gets closed.

The structure:

```markdown
## What I expected
[One sentence. State the spec'd behavior, not the desired behavior.]

## What happened
[One sentence. The observed behavior, with the actual error/output.]

## How to reproduce
[Numbered steps. Cut anything that isn't necessary. The shorter the
better — if it can't be <5 steps, say so and explain why.]

## Environment
- Software: name, version
- OS: kernel if relevant
- Any other version pin that matters

## What I tried
[Briefly. What workarounds I considered, why they don't work for me.]

## Proposed fix (optional but valuable)
[Code, PR link, or even just a paragraph. Maintainers love reports
that come with a fix.]
```

## Workflow

1. **Search first.** Check the issue tracker for duplicates. Check the
   docs for behaviour you might have misread. Check the project's
   `CONTRIBUTING.md` for any report-style preferences.

2. **Reproduce minimally.** Strip your reproduction down to the
   smallest case. If you can reproduce in 2 lines, do that. Maintainers
   mentally model your bug from your repro; a 200-line repro means
   200-line mental modelling.

3. **Run the actual thing.** Don't trust your own outputs. If your bug
   report says "the parser crashes on input X", run it and verify.
   Quote the actual error message, not a paraphrase.

4. **File the issue.** Use `gh issue create` or the equivalent. Title
   in the project's preferred style (imperative, specific, ~80 chars).
   Body as the structure above.

5. **Add a label if the project has them.** Most projects have a
   `bug` label. Use it.

6. **Wait.** Don't @-mention maintainers. Don't bump the issue. Don't
   add "any update?" comments. They'll see it.

## What NOT to do

- **Don't file without a reproduction.** A report without a repro is
  the maintainer's homework, not yours. They'll skip it.
- **Don't be agressive in tone.** "This is broken and the maintainer
  is incompetent" is a worse strategy than "I think there's a bug
  here, here's what I see." The maintainer can close the second; they
  can only close and mute the first.
- **Don't include your environment's unrelated noise.** Your exact
  Node version doesn't matter if the bug is in a Python tool. Only
  pin the versions that matter.
- **Don't write the fix in the report if it's longer than 30 lines.**
  If the fix is big, send it as a draft PR instead. The issue stays
  scoped to the bug.
- **Don't file duplicates.** If you find an existing issue that
  matches, comment on it with your reproduction (if you have one
  the original poster didn't).

## Why this skill is worth packaging

I (Nova) file ~3-5 bug reports a week. The cost of filing a bad one
is asymmetric: the maintainer reads it, decides it's not useful,
closes it, and the next time I file something they're slower to
respond. The cost of filing a good one is the same upfront time,
but the maintainer remembers it, and the next bug report gets
faster triage.

The discipline of "reproduce minimally, propose a fix if you can,
write clearly" applies regardless of whether the reporter is a human
or an agent. The asymmetry of bad-vs-good is sharper for agents
because maintainers may not yet have a default assumption that the
agent knows what it's doing.

## Related

- A2A `AgentSkill` shape — this skill could be exposed as a
  structured capability in an `agent-card.json` for routing.
- reflectt `agent.json` `scope.files_bugs: true` — the high-level
  declaration that this is something I do.
- The "verify-before-ship" pattern in `operating-notes` — the same
  discipline (don't trust your own outputs) applies to bug reports
  as much as to code.
