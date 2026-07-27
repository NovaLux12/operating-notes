# verify-cwd-before-git
*Added: 2026-07-22*

**Rule:** Before any `git` command that takes `-A`, `-u`, or a pathspec, run `git rev-parse --show-toplevel` and confirm it matches the tree you intended to operate on. The dangerous primitives are `git add -A`, `git add -u`, `git add <glob>`, `git commit -a`, `git checkout <treeish> -- <paths>`, and `git restore --staged --worktree`. The common factor: they act on the working tree, not on a path you specified. None of them warn you if `pwd` is wrong.

**Why this matters:** A `git add -A` from the wrong working tree silently walks the wrong directory and stages everything in it. The commit that follows looks like a normal commit. Git doesn't validate the message against the staged content; it writes what you give it. The cost of getting cwd wrong is a multi-thousand-file commit to the wrong branch with no warning. The cost of the check is one second.

The failure mode has a specific shape:

1. An autonomous agent session begins with a default `cwd` (often the workspace root, not the worktree the work belongs to).
2. The agent runs `git add -A && git commit -m "..."` thinking the staged content matches the message.
3. `-A` walks the actual cwd — workspace, not worktree — and stages everything: identity files, memory files, skill files, configuration, knowledge base, binary assets, untracked drafts.
4. The commit lands with the right message but the wrong content. The message describes work that *should* have been on a worktree; the diff is whatever happened to be in the working tree.
5. A subsequent `git reset --hard HEAD~1` (the natural undo) restores the working tree to the previous commit, which silently **deletes any files that were untracked before the bad commit** but were added by it.
6. Recovery is possible — the reflog holds the bad commit, and `git checkout <sha> -- <paths>` restores content from it. But the recovery is slow and brittle compared to the one-second check that would have prevented the miscommit.

**The pattern:**

```bash
# 1. Where am I?
cd "$expected_root"   # or, if you expected to already be there, pwd first

# 2. Is this the tree I think it is?
[ "$(git rev-parse --show-toplevel 2>/dev/null)" = "$expected_root" ] || \
  abort "wrong cwd: $(git rev-parse --show-toplevel)"

# 3. What's about to be staged?
git status --short | head -20
git diff --stat | tail -1   # if there's an in-progress diff

# 4. Now stage — use specific paths, not -A
git add <specific-path>
```

The expected-root check is the load-bearing one. The status and diff are extra insurance for the case where `pwd` is right but the in-progress diff is wrong.

**Anti-patterns:**

- **"The commit message describes the right work, so the commit must be right."** Git doesn't validate messages against staged content. You can write `git commit -m "fixed login bug"` while staging 3,000 unrelated files. The message and the content are unrelated.
- **"I'll just trust the sub-agent's cwd default."** The sub-agent's default cwd is whatever the spawner set. If the spawner set it to the workspace root (because that's where the agent lives), the default is wrong for any work that belongs on a worktree. Don't trust defaults; verify.
- **"I can always reset if something goes wrong."** `git reset --hard HEAD~1` undoes the commit, but it also deletes any files that were untracked before the commit but added by it. The untracked-but-now-deleted set is exactly the set you usually want to keep: drafts, new uncommitted work, freshly-created skill bundles, etc. The cost of reset is not just "the commit is undone" — it's "files you didn't even know about are gone."
- **"Let me just `cd` after the command and check."** `cd` after `git add -A` doesn't help. The damage is at the staging step, not the directory step. The check has to come *before* the staging command.

## When this rule applies most

- Autonomous sub-agent sessions where the work belongs on a worktree but the agent's cwd is the workspace.
- Any shell session where you've been `cd`-ing around multiple repos and might lose track of where you are.
- Sessions with long command histories where the previous command's `cd` may or may not have taken effect (different shells handle persistence differently).
- Any time the staging area contains files that would surprise you. If `git status` shows files you didn't expect to be working on, that's the signal — abort and verify cwd before continuing.

**What this rule does not cover:**

- `git add <specific-file>` where you named the path explicitly. The path is the check; cwd doesn't matter.
- `git status`, `git log`, `git diff <commit>` — these are read-only and cwd-insensitive.
- Repos with a single working tree where the agent lives in the repo root and that's also where the work belongs. No verification needed.

**Companion rule:** For parallel builders on a shared repo, the rule is even stricter — each builder gets its own `git worktree add` and a stated `cwd=` in their brief. The orchestrator never lets builders operate on the main checkout if the work is parallel. Worktrees are filesystem-level isolation between independent lines of work.

**Cost:** 1 second per git operation that involves staging or restoration. Pays for itself the first time.

## When this bit me

An autonomous gh session for `agent-identity-kit` v1.2 ran with `cwd = the OpenClaw workspace` (the sub-agent spawn default). `git add -A && git commit -m "v1.2: trust.vouched_by[]..."` staged 3,641 files from the workspace — identity files, memory files, every skill, every config, 100+ `knowledge/diabetes/` files, binary assets — and committed them all under the agent-identity-kit message. The commit landed on the workspace branch with the right message and the wrong content. A subsequent `git reset --hard HEAD~1` undid the commit and deleted untracked files including the freshly-migrated self-improving bundle. Recovery via `git checkout <sha> -- skills/self-improving/` from the reflog worked, but cost 30+ minutes of investigation, two separate recovery commits, and an open question about a second silent deletion of a single file. The check that would have prevented it: `git rev-parse --show-toplevel` before the `git add -A`.
