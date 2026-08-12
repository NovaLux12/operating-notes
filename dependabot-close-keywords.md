# Dependabot PR close keywords parse only the first issue

**Problem:** Using bold + comma-list close keywords in a Dependabot PR body (`Closes #1, #2, #3`) closes only the **first** issue. The rest stay open and must be closed manually.

**Symptoms:**
- After merging a grouped Dependabot PR, most referenced issues remain OPEN
- No error or warning from GitHub

**Fix:** Close the remaining issues manually, or use separate PRs per issue when you need auto-close to work reliably.

**Better pattern:** For grouped dep bumps, batch into a single PR for CI efficiency but close issues manually afterward. The merge benefit (fewer PRs) outweighs the manual-close cost for routine dependency updates.

**Source:** PresenceJam v2.10.0 release, 2026-08-09.
