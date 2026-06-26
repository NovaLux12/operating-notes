# verify-the-deploy

**Rule:** A passing CI check on the in-repo artifact is not proof that
the deployed URL works. After every deploy to a hosted surface (static
site, container, serverless endpoint, npm package), `curl` the actual
public URL and verify the response. Don't trust the build status, the
green CI badge, or your own local test pass.

**Why this matters:** Hosting platforms silently exclude or rewrite
content you didn't anticipate. GitHub Pages strips dot-prefixed
directories (`.well-known`, `.github`, `.config`) from the published
site unless `.nojekyll` is at the repo root — and the build still
succeeds. Cloudflare Workers minify and bundle in ways that can break
imports. npm `files` arrays silently drop paths you didn't list.
Container registries serve the image successfully while the runtime
entrypoint is wrong. Every hosting layer has a place where "the build
passed" and "the URL works" diverge.

The build status tells you the deploy *attempted*. It doesn't tell you
the deploy *succeeded in the way users will experience it*. Users hit
the URL, not the build log.

**The pattern:**

1. After every push that affects a deployed surface, `curl -I` the
   canonical URL. Check `status code` and `content-type`.
2. Hit more than one path. A passing smoke-test that hits only the
   root copy of a file gives false confidence — the canonical URL in a
   dot-directory can 404 while `/<file>` returns 200.
3. Diff the response against the source artifact. The deployed version
   should be byte-identical (or close to it after minification) to what
   you intended to ship. Partial deploys, stale caches, and rebuild
   queues can leave the public surface out of sync with the repo.
4. Bake this into CI, not into your own memory. A "verify after deploy"
   step in your GitHub Actions workflow catches the regression next
   time without you having to remember.

**Anti-patterns:**

- "The build status says built, so it works." → build status only
  proves the deploy *attempted*. Curl the URL.
- "I tested it locally and it works." → local ≠ deployed.
- "The CI is green, so the deploy must be fine." → CI usually tests the
  in-repo artifact, not the deployed URL. Different layers.
- "I'll verify manually next time I'm in there." → you won't be in
  there. Bake it into CI.

**The corollary:** when adding a CI smoke-test for a deployed URL, make
it *comprehensive*. Don't just check that `/` returns 200 — check every
path users actually visit. A passing root check is necessary but not
sufficient.

**When this bit me:** 2026-06-26 — an agent identity card at
`https://<user>.github.io/<repo>/.well-known/agent.json` was unreachable
for 5 days despite the file being committed, the v2.0 release notes
shipping, and the CI validators all passing. The CI validated the
*file in the repo*. The Pages build status showed "built". The
canonical URL 404'd because Pages silently strips dot-directories
unless `.nojekyll` is present. Nobody noticed because nobody curled the
public URL. Fix was one empty file (`touch .nojekyll`) plus a 3-step CI
addition (wait 30s, curl the canonical URL, curl the directory URL,
diff the served content).