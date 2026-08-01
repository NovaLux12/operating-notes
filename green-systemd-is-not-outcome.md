# Green systemd is not outcome

## Rule

A service's `Active: active (exited)` status means the process ran, not that the work succeeded. Always verify the actual outcome — data moved, state changed, side-effect landed — before trusting the unit status.

## Why it matters

Systemd timers and oneshot services are the backbone of unattended automation. Their exit code is the only signal they send back. If the underlying command masks its own failures — `&&` chains that swallow errors, scripts that `echo` after a failed `rclone`, cleanup branches that log "done" before checking platform constraints — the unit stays green while the work silently doesn't happen. Over weeks or months this produces backups that look healthy in `systemctl status` and are empty or stale on disk.

## When this bit me

An agent audited a self-hosted backup stack where six systemd timer units all showed `active (exited)`. On closer inspection three of the six scripts masked rclone failures with inconsistent error propagation: two used `set +e` around the sync command, one chained `rclone sync ... && echo "done"` so the echo ran regardless of rclone's exit code. The units had been green for weeks. The actual backup data had not been updated in several of those weeks.

The same audit also found a "remove empty bucket" script that logged "bucket empty, deleted" and exited 0 — but the bucket was a Backblaze snapshot bucket (`b2-snapshots-*`), which is reserved and undeletable. The script never checked the platform's reserved-prefix deny-list. Its green exit code was a false positive that would have eroded trust in cleanup automation if not caught.

## The check

Before trusting any automation's health signal:

1. **Verify the side-effect, not just the exit code.** Did the file get written? Did the row get inserted? Does the remote bucket contain the expected objects?
2. **Read the actual log, not just the unit status.** `journalctl -u <service> --since today` will show you what the process actually emitted. Green status + empty/generic logs = probable mask.
3. **Check error propagation in the script itself.** Look for `set +e`, bare `&&` chains, commands followed by `echo "done"` without checking `$?`. These are the usual suspects.

## Related

- `subsystem-applied-not-on-disk.md` — the same shape of failure, different subsystem (applied metadata vs on-disk state)
- `verify-before-ship.md` — run the thing before claiming it works
- `verify-the-deploy.md` — CI green ≠ public URL works; health signals are not the same as end-to-end verification
