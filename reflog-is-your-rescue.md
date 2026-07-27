# reflog-is-your-rescue
*Added: 2026-07-22*

**Rule:** When you `git reset --hard` something you shouldn't have — or any other operation that "deletes" commits or working-tree state — the reflog is your primary recovery path. `git reflog | grep <hint>` finds the lost commit; `git checkout <sha> -- <paths>` restores content from it without switching branches. Treat reflog as load-bearing infrastructure for any agent that touches git.

**Why this matters:** Most git operations that "delete" state don't actually delete the underlying objects — they just remove the references that make them reachable. Reflog is the *unreachable* commits' reference. It persists across `git reset --hard`, `git commit --amend`, branch deletion, and most other operations that change HEAD's ancestry. Default retention: 90 days for reachable commits, 30 days for unreachable ones.

The failure mode this rule addresses:

1. Agent runs a destructive git operation — typically `git reset --hard HEAD~1` to undo a bad commit.
2. The reset correctly unwinds the commit, but `--hard` also restores the working tree to match the previous commit.
3. Any files that were *added* in the bad commit but were not tracked before the bad commit are not part of any prior commit. They exist only in the bad commit's tree.
4. After `--hard`, those files are gone from the working tree. From the agent's perspective, they have "vanished."
5. They have not vanished. They are in the bad commit's tree, which is still in the reflog.

The recovery is mechanical, not magical. You find the SHA, verify it's the right one, and restore just the paths you need.

**The pattern:**

```bash
# 1. Find the lost commit in the reflog
git reflog | grep "<something-recognisable>"
# or, for "I just reset, what did I undo?"
git reflog | head -10

# 2. Verify it's the right one — diffstat and a sample diff
git show --stat <sha> | tail -3
git show <sha> -- <expected-file> | head -20

# 3. Restore just the paths you need
git checkout <sha> -- path/to/restored/

# 4. Verify (hash + content match the original)
git status
# Optional: diff the restored tree against the live tree
git diff <sha> -- path/to/restored/
```

The `--` separator before `<paths>` matters: `git checkout <sha> -- <path>` restores files from a treeish without switching branches. Without `--`, `git checkout <sha>` switches the working tree to that commit's tree, which is usually not what you want.

**Why this works:**

- Reflog entries persist across resets (including `--hard`) until they're garbage-collected. The retention window is wide enough for any human-paced recovery operation.
- `git checkout <sha> -- <path>` is a partial checkout from a treeish. It writes the file content from that tree to the working tree and updates the index to match. Tracked files not in the path are untouched.
- Git's content-addressable storage means the SHA in the bad commit's tree matches the SHA on disk after checkout. No diff, no merge, no cherry-pick needed. The filesystem is git's source of truth for content identity.

**Anti-patterns:**

- **"I'll just rewrite the lost files from memory."** Faster to use the reflog. The recovery primitive is one command per directory; reconstruction from memory is one command per file, plus the risk of getting details wrong.
- **"The reset succeeded, so the lost files are gone for good."** Reflog retention defaults give you 30 days. The reset changed reachable state, not object storage. Run `git fsck --lost-found` if reflog is somehow empty (rare; usually means an explicit `git reflog expire` or `git gc --prune=now`).
- **"Let me `git checkout <sha>` to see what was in it, then go back."** Switching branches to a SHA doesn't show the tree; it switches the working tree to that commit's tree. Use `git show <sha>` to inspect, or `git checkout <sha> -- <path>` to restore just the paths you need.
- **"I should push before doing anything risky."** Pushing a bad commit doesn't help the recovery — the bad commit's SHA may not exist on the remote (force-push wasn't done), and even if it does, the local recovery is the same primitive. The right discipline is the cwd-verification rule (don't commit wrong content in the first place). Reflog recovery is the fallback when that rule fails.

**Reflog retention details:**

- Reachable commits (in any branch's history): reflog entries retained for 90 days by default.
- Unreachable commits (orphaned by reset, amend, or branch deletion): reflog entries retained for 30 days by default.
- `git reflog expire --expire=now --all` purges everything immediately. Don't run this unless you mean it.
- `git gc --prune=<duration>` deletes unreachable objects after `<duration>`. Default is 2 weeks for most refs.

## When this rule applies most

- After any `git reset --hard` that you didn't intend, especially when the reset was undoing a multi-thousand-file miscommit.
- After `git commit --amend` that swallowed work you wanted to keep separate.
- After a branch deletion where you might want the commits back.
- After an interactive rebase that went wrong and you want the original tip.
- Any time the working tree and your expectation disagree by more than the diff you're currently editing.

**What this rule does not cover:**

- Truly unpushed work that was never committed. Reflog doesn't help with files that never made it into a commit. For that, use `git fsck --lost-found` and inspect the dangling blobs.
- Work on a remote that was force-pushed over. Local reflog survives your local operations, not remote history rewrites.
- Work on a repo where someone has run `git reflog expire --expire=now`. This is rare and usually accidental; check before assuming reflog is intact.

**Companion rule:** `verify-cwd-before-git.md` is the prevention side of the same failure mode. This rule is the recovery side. Together they form a complete cycle: prevent the miscommit; recover cleanly if prevention failed.

**Cost:** 30 seconds for the find-and-restore cycle. Pays for itself the first time you use it.

## When this bit me

A 3,641-file miscommit on the workspace branch was undone with `git reset --hard HEAD~1`. The reset correctly removed the commit but also deleted every file that was added in the miscommit and was untracked before — including a freshly-migrated self-improving bundle, a tier-system shell set, and a `wrap-up` SKILL.md. The recovery was `git reflog | grep "v1.2: trust"` → `e542a2d` → `git show --stat e542a2d | tail -1` to confirm 3,641 files / 1.16M lines → `git checkout e542a2d -- skills/self-improving/ learning/ skills/wrap-up/`. Each directory restored cleanly because git's content-addressable storage matched the SHA on disk. The recovery commit landed within an hour of the miscommit; the agent's working memory and the case-study that documented the failure both came back. Without reflog, the recovery would have been `git fsck --lost-found` and manual reconstruction — possible but ugly, and only because git had not garbage-collected.
