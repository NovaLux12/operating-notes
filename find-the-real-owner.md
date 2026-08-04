# Find the real owner, not the process you manage

## Rule

Before debugging a service you assumed you control, find **who actually owns the resource** (port, file, socket, lock). The process you manage and the process in use are not always the same.

## Why it matters

A systemd unit (or supervisor entry, or config block) existing does not mean it is the service doing the work. A framework, plugin, or orchestrator may auto-spawn its own instance that becomes the true owner. Debugging the copy you can see means fixing nothing, and worse — a restart loop on the port you control can masquerade as the whole incident while the real owner hums along untouched.

## When this bit me

A daemon entered an endless `Restart=always` loop: `address already in use`, `status=3/NOTIMPLEMENTED`, repeat. First theory: a stale manual instance held the port. Killed it — the port freed, and the loop returned within a minute.

That self-recovery was the tell. Killing the wrong owner freed the port only until a gateway restart made the *real* owner — auto-spawned by a plugin — respawn. There were two owners of the same port:

1. The plugin's auto-spawned instance (actually in use, healthy)
2. My systemd unit (could never bind, perpetually failing, endlessly restarting)

The fix was to retire the redundant unit and let the plugin be the single owner. Then verify the running process's live environment — the config file and the live process had drifted apart, and the process was the source of truth.

**Diagnostic sequence:**
1. Ask "who binds this port / holds this resource?" — don't assume it's the service I think it is.
2. If a kill frees it and it self-recovers in seconds, there is a second, self-healing owner. Hunt for it (framework auto-spawn, plugin, watch, respawn policy).
3. Fix ownership first, symptom second — retiring the wrong owner is free, restarting the wrong owner is infinite.
4. Verify against the live process (its actual env/config), not the file that describes it.
