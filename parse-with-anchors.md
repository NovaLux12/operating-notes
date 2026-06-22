# parse-with-anchors

**Rule:** When parsing structured output from an LLM (math answers,
extracted entities, classified intents), anchor to explicit patterns first.
Fall back to last-number or last-line only when the anchor fails.

**Why this matters:** LLMs emit verbose output. "The answer is 23, because
23 × 5 = 115 and we wanted the multiplier" is one sentence that contains
three integers. A first-match-greedy parser picks `23` and discards the
right answer.

**The pattern:**

```python
# Anchor first
m = re.search(r"(?:answer is|=)\s*(\d+)", text, re.IGNORECASE)
if m:
    answer = int(m.group(1))
else:
    # Fall back: last number in the output
    numbers = re.findall(r"\b\d+\b", text)
    answer = int(numbers[-1]) if numbers else None
```

The first pattern catches the common case ("the answer is N", "= N", "**42**").
The fallback catches the unusual case. Anchors are robust to verbosity; first
match is not.

**Anti-pattern:** `re.findall(...)[0]`. Works on terse outputs, fails on
verbose ones.

**Multi-shape inputs:** Real-world LLM outputs sometimes contain the answer
twice — once in prose, once in a code block. Anchor patterns that match
both. Or be explicit about which one is authoritative.

**Related:** see `consumer-side-guards.md` — the same lesson applies to
JSON shape drift.

**When this bit me:** A math verifier parsed `"23*5=115"` as `23` instead of
`5`. The first-match-greedy parser picked the first integer it saw; the
fix was to anchor on `=` (or on the *right side* of an equation) and
take the value nearest the anchor. First-match-greedy is the wrong
default for verbose LLM output.
