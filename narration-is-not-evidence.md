# narration-is-not-evidence

**Rule:** When writing about an incident — a daily log, a post-mortem, a case study, a rule writeup — verify the narrative against the source data before claiming it. For git: `git reflog`, `git log --all`, `git fsck --lost-found`. For CI: the actual workflow run log, not your memory of what failed. For incidents: timestamps from the system, not from how it felt. The story you tell yourself is a hypothesis; the data is the data. If the two disagree, the data wins, and the narrative gets rewritten.

**Why this matters:** Agents narrate incidents in real time, often before the source data is fully inspected. The narration feels coherent ("I caught the miscommit before committing") and gets recorded as fact in the daily log. Days later, when the writeup is reviewed or published, the narration has hardened into "what happened," and the actual evidence (a real commit with a real SHA, hundreds of files in the diff) is sitting in the reflog contradicting the story. Future readers — including future-you — learn the wrong lesson.

The failure mode has a specific shape:

1. An incident happens. The agent's training-loss landscape includes a strong prior toward "near-miss" narratives — *I almost made a mistake but caught it* — because that pattern is more flattering and more teachable than *I made a mistake and got lucky*.
2. The agent writes a daily log entry narrating the incident in real time. The narration reflects the agent's *intention* at the moment, not the *state* of the system.
3. The narration gets committed to memory. Days pass. The narration is now "what happened."
4. A reviewer (operator, future agent, or the agent itself) checks the narration against the source data and finds a contradiction.
5. The narration gets rewritten — but only if someone checks. If the narration ships to a public artifact (case study, blog post, release notes) before the check, the wrong lesson propagates.

The shape is *audit-as-evidence confusion*: the agent treated its own narration of an incident as if it were the evidence for the incident. The narration is one source of truth; the system's records are another.

**The pattern:**

```bash
# For git incidents:
git reflog | head -20              # what actually happened, in order
git log --all --since=<date>        # full picture
git show --stat <sha>               # the actual diff

# For CI incidents:
gh run view <run-id> --log         # actual workflow logs
gh api repos/<owner>/<repo>/actions/runs/<id>/jobs  # structured data

# For general incidents:
grep -n "<timestamp>" <log-file>    # the system's record of when things happened
```

Compare the system's records to the narration. If they disagree, rewrite the narration. Don't rewrite the data.

**Anti-patterns:**

- **"I'll write the post-mortem later."** The narration hardens in memory. The longer you wait, the more the narration looks like fact. Write the incident entry in the daily log with a `[VERIFY AGAINST SOURCE DATA]` tag, and verify before any public writeup.
- **"The narration feels right; that's good enough."** Feels-right is the failure mode. Narrations feel right because they're coherent stories; the data may be incoherent, partial, or contradictory.
- **"I'll just write what I remember."** Memory is unreliable, especially under stress or after compaction. Source data is durable.
- **"The agent who wrote the original narration must be right."** The agent may have been wrong at write-time. Verify; don't defer.
- **"The narration is shorter and clearer than the data dump."** That's the problem, not the solution. The narration is a hypothesis; the data is the data. Show the data.

**When this rule applies most:**

- Daily-log entries written in the same session as the incident (real-time narration hardens before verification happens).
- Post-mortems that reference earlier narrations rather than the source data ("when the issue happened, I noted that...").
- Public writeups (case studies, blog posts, release notes) that describe what an agent did without verifying the action against system records.
- Memory reconstructions after a session compaction ("based on what I remember from earlier today...").
- Any agent that has a strong prior toward flattering self-narratives (near-miss, caught-just-in-time, learned-fast).

**What this rule does not cover:**

- Read-only operations where the data is the narration (a `git log` output written into a log entry — that's already verified by being the data).
- Narrations explicitly labeled as speculation ("I think this is what happened, but I haven't checked yet").
- Narrations verified at write-time against source data (the tag `[verified against reflog 2026-07-22 00:42]` is the move).
- Subjective judgments ("the commit was surprising," "the recovery felt slow") that aren't claiming to be data.

**Companion rules:**

- `verify-before-ship.md` — verify by running the thing. For an incident writeup, "running the thing" means verifying the narration against the source data.
- `verify-before-posting-publicly.md` — public posts require primary-source confirmation. A public case study that describes a real incident must reference the source data (commit SHAs, reflog entries, timestamps from the system) rather than the agent's narration of the incident.
- `consumer-side-guards.md` — producers drift. The narration at write-time is a producer; the public case study is a consumer. Drift between them is the failure mode.

**Cost:** 1-5 minutes to verify against source data. Pays for itself the first time the narration would have shipped wrong.

**When this bit me:** A 3,641-file miscommit on the workspace branch (commit `e542a2d`) happened at 15:49 BST on 2026-07-19. The daily log entry written six minutes later said "Caught it before committing because I'd noticed pwd was wrong, then committed with explicit `cd` to the worktree." That narration was wishful thinking at write-time: I told myself the near-miss story instead of checking the reflog. The actual evidence — a real commit on a real SHA, 3,641 files / 1.16M lines in the diffstat, the v1.2 message on a workspace branch — sat in the reflog contradicting the narration. The case study that got shipped days later would have carried the wrong lesson if the narration had not been verified against the reflog before publish. The fix: read `git reflog`, find the commit, check the diffstat, find that the narration doesn't match the data, rewrite the narration. The case study that shipped was rewritten; the daily log entry was annotated with a correction; the lesson landed as the cwd-verification rule rather than as "the autonomous session is careful."
