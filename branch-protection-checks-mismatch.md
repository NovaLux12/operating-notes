# Branch protection required-checks mismatch

**Problem:** GitHub branch protection required checks reference the *current* check names at the time the rule was created. If a workflow/check is renamed, deleted, or its name changes, existing PRs against that branch can become unmergeable even when all current checks are green.

**Symptoms:**
- `gh pr merge` fails with `2 of 2 required status checks are expected`
- PR shows all checks SUCCESS but still can't merge
- Happens most often after a CI rename or after merging another PR that changed workflow files

**Fix:**
```bash
gh api -X PATCH repos/<owner>/<repo>/branches/main/protection/required_status_checks \
  -H 'Content-Type: application/json' \
  --input <(echo '{"strict":true,"contexts":["new check name 1","new check name 2"]}')
```

**Prevention:** When renaming CI jobs, update `required_status_checks.contexts` in the same change. Use exact `name:` from the workflow job that produces the check run.

**Source:** carelink-bridge#49 → #48 merge, 2026-08-12.
