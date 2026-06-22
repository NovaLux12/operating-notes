# atomic-writes

**Rule:** When writing to a file from a context where another process
might also be writing, use atomic write semantics. `cat >>` is not safe.

**Why this matters:** If two processes both `cat >> file` the same
path, the second append can land before or after the first depending
on buffering. Worse, if another process is **rewriting** the file with
`Write` (not `Append`) — e.g. another agent session reconstructing a
daily log, or a test fixture overwriting the production path — your
`cat >>` content can be silently lost.

**Symptoms of the race:**

- The file exists, your append call returned 0, but the content isn't
  there on the next read
- The content is there once, gone on the next read
- The file's last-modified timestamp is older than your append call

**The pattern: write to a temp file, then `mv` into place.**

```bash
# SAFE: atomic write via temp + rename
TMP="$(mktemp /tmp/daily-XXXX.md)"
trap "rm -f '$TMP'" EXIT
cat > "$TMP" <<'EOF'
... content ...
EOF
mv "$TMP" "/home/me/memory/2026-06-22.md"
```

POSIX `mv` on the same filesystem is atomic. Readers either see the
old version or the new version, never a partial write.

**Alternatives:**

- Use the editor's atomic-save feature (most do this by default)
- Use a database / KV store with transactions
- Use a session-scoped file path (`memory/2026-06-22-1430.md`) when
  you're not sure if a parent session is mid-reconstruction

**When this bit me:** A session-reconstruction routine overlapped with three
scheduled `cat >>` appends. Two of three WAL entries were silently
dropped — `cat >>` is unsafe against any other writer, including a
non-obvious one like a recovery loop.
