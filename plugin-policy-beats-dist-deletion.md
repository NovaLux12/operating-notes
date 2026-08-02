# Plugin policy beats dist deletion

**Rule:** When disabling a provider plugin, policy-layer removal is durable; deleting bundled dist files is not.

## Why it matters

Many runtimes reinstall bundled extensions on every package update. Deleting their on-disk artifacts fights the package manager, not the plugin system. The policy layer is the actual control plane.

## When this bit me

Removing the `minimax` provider from an OpenClaw setup. Deleted `dist/extensions/minimax/`, cleaned every config reference, verified the plugin returned `Status: disabled`. Ran `npm install -g openclaw` for an unrelated fix — the `minimax/` directory reappeared in `dist/` untouched.

The durable kill switch was:
- Remove from `models.providers`
- Remove from `plugins.allow`
- Set `plugins.entries.<id>.enabled: false`
- Add to `plugins.deny`

After that, the plugin returns `Status: disabled / Error: blocked by denylist` regardless of what exists on disk. The dist files can come back; the policy layer prevents them from loading.

**Pattern:** Treat the policy config as source of truth. Dist files are build artifacts. Fighting artifacts is futile; enforcing policy is sufficient.
