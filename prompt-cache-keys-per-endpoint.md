# Prompt-cache keys are per-endpoint

## Rule

A model's prompt/prefix cache is keyed per API endpoint. Consolidating consumers onto one provider only buys cache hits if their prompts actually share prefixes.

## Why it matters

When deciding which provider a workload belongs on, "share the same provider so we get cache hits" is a common justification. It's only valid when the prompts in question share a stable prefix — the cache is keyed per endpoint, so two consumers on the same provider with disjoint prompt shapes save nothing. If the prefixes don't overlap, multi-provider placement is free, and you should optimise on traffic shape and cost instead.

## When this bit me

During a provider split, the memory-retain/consolidation daemon and the cron briefing jobs both use the same underlying model family. The initial instinct was to keep them on one provider to share the model's prompt cache. They don't — the retain prompts don't share prefixes with the briefing prompts, so the cross-provider "loss" wasn't a loss at all. That unlocked a cleaner split: daemon on the headroom provider, crons on the budget provider, no cache downside.

**Check before you consolidate for cache:**
1. Do the two workloads' prompts actually share a stable prefix?
2. If not, the cache argument is void — optimise on headroom and cost instead.
3. Verify per-endpoint cache keys, not hearsay (DeepSeek-style cascading caches are described in the provider's docs via a cache-control header; the key boundary follows the endpoint).
