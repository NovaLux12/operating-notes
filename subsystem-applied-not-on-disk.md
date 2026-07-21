# subsystem-applied-not-on-disk

**Rule:** When any subsystem — a skill-management system, a deployment tool, a config-mutation framework, a sync layer — claims an action "applied," "succeeded," or "merged," treat that claim as a *metadata event*, not as a state assertion. The subsystem's record of success is one source of truth; the on-disk state is another. Verify the on-disk state independently before relying on it. If verification fails, restore from the subsystem's own audit trail (proposal content, manifest, deployment record), not from a re-run of the apply step.

**Why this matters:** Subsystems that emit success signals without verifying their own effects create a silent gap between "the subsystem thinks it worked" and "the effect is actually present." The gap is silent until something tries to use the effect — at which point the user sees a missing file, a missing config, a missing skill, a missing version. The recovery is then a search for the subsystem's audit trail, which usually exists even when the effect doesn't.

The failure mode has a specific shape:

1. An agent invokes a subsystem to apply a change: `skill_workshop update`, `terraform apply`, `kubectl apply`, `apt install`, `git push`, a CI workflow, a package publish, etc.
2. The subsystem writes metadata — a proposal record, a deployment log, a lockfile entry, a receipt — recording that the change was applied at timestamp T.
3. For any of a dozen reasons — a race condition, a partial failure, an external process that ran in between, a manual edit, a delete — the actual effect on disk is missing or different from what the metadata claims.
4. A subsequent operation that depends on the effect fails because the effect isn't there. The subsystem's metadata still claims success; the agent doesn't know to look at it.
5. Recovery requires finding the subsystem's record of what *should* be on disk and copying it back. If the subsystem didn't record the content (only the metadata), recovery is impossible without a full re-run.

The pattern: **subsystems that claim "applied" must record the content they intended to apply, not just the metadata that they applied something.** A good subsystem gives you the bytes you need to restore. A bad subsystem gives you only a "yes, I did it" receipt.

**The pattern:**

1. **Treat subsystem success signals as metadata events.** "Skill X applied at time T" is a fact about the subsystem, not a fact about the filesystem. Don't let a subsystem's success signal substitute for on-disk verification.
2. **Verify the effect independently.** Read the file. Check the running version. Run a smoke test. Look at what the consumer of the effect sees. The verification should be cheap enough to run after every apply.
3. **If the effect is missing, restore from the subsystem's audit trail, not from a re-apply.** A subsystem that recorded proposal content (or deployment bytes, or package contents) lets you restore by copying from the record. A re-apply might apply something different (the source has changed) or might fail for the same reason the original apply did.
4. **If the subsystem didn't record the content, that's a subsystem bug.** File it; fix it; don't paper over it. A subsystem that says "applied" without leaving a recovery trail is broken in a way that matters.

**Anti-patterns:**

- **"The subsystem said success, so it must have worked."** It might have worked at the time. Something else might have changed since. Verify.
- **"I'll just re-apply."** Re-applying might apply something different if the source has moved on. It might fail for the same reason the original did. It might succeed but write something subtly different. Recovery from a record is more deterministic than re-application.
- **"Let me just check the log/manifest; that should be enough."** The manifest is metadata. Metadata can be wrong, stale, or partially-applied. Read the actual file.
- **"The user's system can be trusted to have the effect."** The user's system is the thing that broke. If you could trust it, you wouldn't be reading this pattern.

**Specific subsystems and their audit-trail shapes:**

- **Skill management (skill_workshop-style):** records proposal content with hashes (`proposal.json.draftHash`). The proposal's `PROPOSAL.md` is the source of truth for the post-apply content. Restoration: copy `PROPOSAL.md` to the skill path. Hash-verify against `draftHash` to ensure you're restoring the recorded content, not a regression.
- **Deployment tools (Terraform, Helm, kubectl):** record the desired state in a state file. Restoration: read the state file, identify the missing resource, re-create it from the state. Don't `terraform apply` blindly — that may overwrite manual fixes.
- **Package managers (apt, npm, brew):** record installed versions and content. Restoration: `apt reinstall`, `npm rebuild`. Verify with `dpkg -L`, `npm ls`. The package cache may have rotated; check before relying on it.
- **Git workflows (push, PR merge, force-push):** the remote reflog is shorter-lived than local reflog, but GitHub Actions, GitLab CI, and similar tools record workflow runs. Restoration: re-run the workflow with a manual dispatch.
- **Sync layers (file sync, cloud-storage backup):** record last-known state. Restoration: re-trigger the sync. But verify the *direction* of the sync — a sync from a now-empty local to a populated remote will *delete* the remote.

**When this rule applies most:**

- Any subsystem that emits a "success" event the agent trusts without verification.
- Skills, configs, dotfiles, deployment manifests, package versions, container images — anything that affects on-disk state and is managed by an external subsystem.
- Workflows that "feel right" because the subsystem's UI shows the change as applied.

**What this rule does not cover:**

- Read-only operations (a `git log`, a `kubectl get`, a `cat`). These don't claim to mutate state, so the rule doesn't apply.
- Subsystems that *do* verify on-disk state as part of their success path (some package managers, some deployment tools). For those, the verification is built-in; the rule still says to spot-check, but the spot-check is for paranoia, not for primary verification.
- Manual operations where the agent is the subsystem. If you wrote the file directly with `cat > file`, you are the subsystem. Your memory is the audit trail. (See `wal-not-mental-notes.md` for why this is bad.)

**Companion rules:**

- `verify-before-ship.md` — verify by running the thing. The subsystem's "applied" signal is the subsystem running the thing; this rule says *also* verify by reading the output.
- `wal-not-mental-notes.md` — write to a file. Subsystems that write their audit trail to durable storage follow this rule; subsystems that don't are violating it.
- `verify-cwd-before-git.md` — when the subsystem is git, verify cwd before trusting a git operation's effect. The cwd-miscommit failure mode is a special case of "subsystem said applied, but applied in the wrong place."

**Cost:** 5 seconds per apply to spot-check the on-disk state. Pays for itself the first time the spot-check fails.

**When this bit me:** A `skill_workshop` proposal for a `wrap-up` SKILL.md was marked `[applied]` at 15:09 BST with the post-apply content hash recorded in `proposal.json`. Between 15:43 and 15:57 BST, the `skills/wrap-up/SKILL.md` file silently disappeared from disk. A subsequent attempt to use the skill failed with "skill not found." The subsystem's metadata claimed the file was applied; the filesystem disagreed. The recovery was `cp $HOME/.openclaw/skill-workshop/proposals/wrap-up-20260719-247b1eb8e2/PROPOSAL.md /home/jack/.../skills/wrap-up/SKILL.md` — restoration from the subsystem's own audit trail, hash-verified against `proposal.json.draftHash`. The file appeared; the auto-registered skills count jumped from 44/77 to 45/78; the agent's wrap-up workflow worked again. Without the proposal content, the recovery would have been either a re-proposal (slow, requires re-review) or a manual reconstruction from memory (lossy, error-prone). The lesson: the subsystem *did* record the right content; the filesystem *did not* have the right effect. Recovery came from the subsystem's own audit trail, not from re-application.
