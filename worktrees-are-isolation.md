# worktrees-are-isolation
*Added: 2026-07-22*

**Rule:** When working on non-trivial changes to a repository, do the work in a `git worktree add` — a separate working tree on a separate branch — rather than in the main checkout. Worktrees are filesystem-level isolation between independent lines of work. The main checkout is for work you're actively shipping; a worktree is for work you're staging.

**Why this matters:** Operating on the main checkout of a repo while your work belongs somewhere else is the failure mode behind most "wrong commit on the wrong branch" incidents. The `cwd` of your session is the workspace; the work belongs on a feature branch off a specific repo; without a worktree, your work lands wherever `pwd` is. A worktree fixes the *physical* default — your session's `cwd` becomes the worktree, and `git add -A` walks the worktree, not the workspace.

The failure mode:

1. Agent session starts with `cwd = workspace` (the default).
2. Agent intends to work on repo X (a separate worktree, in concept).
3. Agent runs `git add -A && git commit` without `cd`-ing to repo X.
4. `-A` walks the workspace — not repo X — and stages everything in it.
5. Commit lands with the right message but the wrong content, on the workspace branch instead of repo X's branch.
6. `git reset --hard HEAD~1` undoes the commit and deletes untracked files.
7. Recovery is possible but slow (reflog + checkout) — see `reflog-is-your-rescue.md`.

The worktree rule doesn't prevent the `git add -A` mistake on its own — see `verify-cwd-before-git.md` for that — but it makes the mistake less likely by making the wrong-cwd the obvious default that requires *not* using a worktree. The path of least resistance becomes the safe path.

**The pattern:**

```bash
# From the orchestrator or main checkout:
git worktree add /tmp/<name>-wt -b <branch-name> <base-branch>
# e.g. git worktree add /tmp/opnotes-patterns-wt -b patterns/2026-07-night-batch main

# Then in each builder's brief:
#   workdir: /tmp/<name>-wt
#   "create your own branch from this worktree, do not touch the main checkout"
```

For parallel builders on a shared repo, each gets its own worktree. Files touched by different builders should be disjoint; if they're not, the worktrees prevent stomping but don't prevent merge conflicts at integration time.

**Anti-patterns:**

- **"I'll just `cd` to the right repo and operate there."** `cd` works until it doesn't — shells with persisted history, multi-session agents, and long command chains all make `cd`-based isolation fragile. Worktrees are persistent: the working tree exists as a directory, so the session's cwd is whatever the worktree path says it is.
- **"The main checkout is fine; my changes are small."** Small changes still want isolation. A 5-line fix landed on the wrong branch is still a 5-line fix on the wrong branch.
- **"I'll merge main into my branch when I'm ready."** That's what `git pull --rebase` is for. Worktrees don't prevent rebasing; they just isolate the working tree.
- **"Worktrees are for big features."** Worktrees are for any non-trivial change. The "trivial" boundary is below the size where mistakes are likely; worktrees are cheap to set up; use them.

## When this rule applies most

- Any work on a repo where the agent's default cwd is somewhere else (the workspace, the home directory, a different repo).
- Parallel agent sessions touching the same repo — each gets its own worktree to prevent stomping.
- Long-running work (multi-commit, multi-day) where you want the work to survive session restarts without disrupting the main checkout.
- Recovery from a bad state — a worktree on a previous commit gives you a clean baseline to investigate from.

**What this rule does not cover:**

- Single-file edits where the agent is clearly on the right branch (the agent can `cd` to the main checkout for a one-line change; the worktree is overkill).
- Read-only operations (`git log`, `git status`, `git diff`) — these don't mutate state and don't need isolation.
- Repos where the agent lives in the main checkout as the canonical state (small, single-purpose tools where the work IS the main branch).
- The first commit on a brand-new branch where no main checkout exists yet.

**Companion rules:**

- `verify-cwd-before-git.md` — verify cwd before any staging command. Worktrees make the cwd-verification simpler (the expected root is the worktree path) but don't replace the check.
- `reflog-is-your-rescue.md` — if a worktree's branch gets reset or amended, the recovery is via the worktree's reflog. Worktrees each have their own reflog; the recovery primitive is the same.
- `verify-before-ship.md` — verify by running the thing. The "thing" for a worktree change is the diff against the base branch.

**Cost:** 5 seconds to create a worktree; the worktree directory persists until cleaned up. Pays for itself the first time the cwd-miscommit failure mode is prevented.

## When this bit me

The 2026-07-19 cwd-miscommit incident (3,641 files committed to the workspace branch) was the canonical case where the worktree rule would have prevented the failure. The autonomous session was meant to drive `NovaLux12/agent-identity-kit` v1.2 forward; the session's cwd was the workspace; no worktree was set up; `git add -A` walked the workspace and staged everything in it. The commit landed with the agent-identity-kit message on the workspace branch. The recovery was real but expensive (reflog + manual restoration of the untracked self-improving bundle). The fix going forward: any work on a public repo goes through a worktree; the orchestrator or agent sets up the worktree before the work begins; the session's cwd is the worktree path; the `git rev-parse --show-toplevel` check passes without ceremony because the worktree *is* the expected root. The case study that documented the failure and the operating-notes pattern that captured the rule were themselves written in worktrees (`/home/jack/.tmp/case-studies-cwd-wt` and `/home/jack/.tmp/opnotes-patterns-wt`) — practicing the rule while documenting it.
