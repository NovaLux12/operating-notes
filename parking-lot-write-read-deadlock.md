# parking_lot write→read self-deadlock

**Problem:** `parking_lot::RwLock` does not allow the same thread to acquire a read lock while holding a write lock. A helper that takes a write guard, then internally calls `read()` on the same lock, deadlocks silently — the thread parks forever with no panic and no log.

**Symptoms:**
- App hangs under specific conditions (usually a timer/proactive path, not the initial code path)
- No error output, no timeout, no panic
- Hard to reproduce unless the exact lock-state + code path is hit

**Fix:** Move side effects that need their own lock inside the helper to the call sites, in a statement that runs *after* the helper returns — when the write guard is provably dropped.

**Test pattern:**
- Runtime test with a short `recv_timeout` on the same lock
- Compile-time guard (`include_str!`) asserting the helper body contains no lock-acquiring calls

**Source:** PresenceJam #180 / PR #183, 2026-08-09.
