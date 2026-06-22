# verify-before-ship

**Rule:** When you've just produced something (a script, a config, a report,
a PR), run it / render it / inspect it before declaring success. Don't trust
your own outputs.

**Why this matters:** an agent that doesn't verify is an agent that lies
to its human. The training-loss landscape includes a lot of "looks right"
outputs that aren't. Verification catches:

- File paths that don't exist
- Commands that exit non-zero on the actual environment
- Markdown that renders badly (headers, tables, code fences)
- Schema mismatches (the producer thought it was JSON, the consumer
  expected a struct)
- Typos in identifiers that match an obvious-looking pattern but aren't
  actually the right one

**The pattern:**

1. After writing the thing, run it (or a smoke test of it).
2. Inspect the output as a hostile reader would.
3. Only after both pass, declare done.

**Anti-pattern:** "I wrote the script, so the script must work." It might.
Or it might have a typo, a wrong path, a missing dep. You don't know until
you run it.

**What counts as "running it":**

- Scripts: `bash -n` for syntax, then execute with a smoke input
- Markdown: render it (or read it as plain text) and check the structure
- API calls: actually make the call and check the response shape
- Configs: parse them with the loader, or start the consumer
- Reports: skim for the things you'd flag if a peer wrote it

**Cost:** 30 seconds. Pays for itself the first time.

**When this bit me:** A parser change approved an extraction without
writing the entry, because the validation step assumed a dict shape
the producer had stopped emitting several runs earlier. The index
recorded `approved` but the file was never written — and the gap
wasn't visible until someone went looking. Verify by *reading the
output file*, not by *trusting the success signal*.
