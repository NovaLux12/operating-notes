# consumer-side-guards

**Rule:** When writing downstream code that consumes an extraction or
parsing pipeline, put the guard on the **consumer** side, not the producer
side. Producers drift; that's a feature, not a bug to "fix".

**Why this matters:** An LLM producer will eventually return a different
shape than the one you tested with. A dict becomes a string; a list
becomes a string-list; a field disappears; a new field appears. If your
consumer crashes on shape change, your whole pipeline stops, and you
won't notice for days because the failure is silent.

**The pattern:**

```python
# WRONG: assumes shape
author = post["author"]["name"]

# RIGHT: handles multiple valid shapes
a = post.get("author")
if isinstance(a, dict):
    author = a.get("name", "unknown")
elif isinstance(a, str):
    author = a
else:
    author = "unknown"
```

A three-line guard at the consumer prevents a multi-day debugging session
when the producer shape drifts.

**Anti-pattern:** "I'll add a comment that says `author is always a dict`."
It isn't. It wasn't always; it won't always be.

**Test fixtures should reflect worst-case API shape, not convenient shape.**
If the real API sometimes returns a dict and sometimes a string, your
tests should cover both. A test that only covers the convenient shape
will pass while the production pipeline silently fails.

**The deeper rule:** The producer is a thing you don't fully control —
even when it's you, because you don't control what you'll do next week.
The consumer is the thing you fully control. So put the burden there.

**When this bit me:** 2026-06-20, three daily cron runs crashed on the
same extraction. The LLM had started emitting
`people: ["Amy Hodges"]` instead of
`people: [{"name": "Amy Hodges", "role": null}]`. The consumer assumed
the dict shape and crashed. The index marked it `approved` before the
crash, so the data-integrity bug hid for three days.
