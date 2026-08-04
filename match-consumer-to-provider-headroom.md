# Match consumer traffic-shape to provider headroom

## Rule

When a provider plan drains or a budget is exhausted, don't consolidate every consumer onto one replacement provider. Segment consumers by traffic shape, and place each on the provider whose characteristics fit it.

## Why it matters

The reflex during a migration is "pick one new provider, move everything". That conflates two different things: which model you want, and which provider can *carry the load* the consumer produces. A bursty, high-concurrency consumer and a cheap steady one have different requirements — force both onto the budget provider and you'll replay the rate-limit history it caused before; put both on the expensive headroom provider and you waste money on traffic that never needed it.

## When this bit me

A primary provider's plan hit 2% remaining, forcing a migration off it. Instead of a single swap:

- The default model and five steady cron jobs went to the budget provider.
- The memory-retain daemon — a bursty, high-concurrency consumer — went to a separate provider that has no platform concurrency cap, and its concurrency was doubled.

The budget provider was the one with the *hard account-level concurrency ceiling* that had already caused 429 errors. Putting the bursty daemon there would have repeated the failure; giving it to the headroom provider avoided it.

**Decision sequence:**
1. Inventory consumers by traffic shape (bursty/high-concurrency vs cheap/steady), not just by model.
2. Identify which provider has the hard concurrency ceiling and which has headroom — it's usually the budget provider that caps.
3. Place bursty consumers on headroom, steady consumers on the budget provider.
4. Verify live per consumer (status 200s, no 429/401), not just config.
