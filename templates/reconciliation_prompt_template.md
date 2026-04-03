# Reconciliation Prompt Template

This template is for the synthesis step after you have collected parallel responses from your council. You feed the independent outputs back into a model (often a different one) to identify consensus, flag disagreements, and produce a single unified recommendation.

## Primary Reconciliation Template

```
I have collected independent analyses from [N] AI models on the following question:

ORIGINAL QUESTION: [paste or summarize]

MODEL A RESPONSE (SUMMARY):
[Key points, recommendation, reasoning]

MODEL B RESPONSE (SUMMARY):
[Key points, recommendation, reasoning]

MODEL C RESPONSE (SUMMARY):
[Key points, recommendation, reasoning]

[MODEL D RESPONSE if applicable]

YOUR TASK:
1. Identify where the models AGREE (high-confidence signals)
2. Identify where the models DISAGREE (needs investigation)
3. Flag any claims that appear unsupported or potentially hallucinated
4. Synthesize the strongest elements from each response into a single, unified recommendation
5. Note any tradeoffs or caveats the final recommendation should acknowledge
6. Rate your confidence in the synthesis: HIGH / MEDIUM / LOW
```

## Triple-Audit Verification Template

Use this after reconciliation to stress-test the synthesized output. Send it to a model that was not involved in the original council if possible.

```
Here is a synthesized recommendation based on multiple independent analyses:

SYNTHESIZED RECOMMENDATION:
[paste the reconciled output]

YOUR TASK:
- Challenge this recommendation. What did we miss?
- What assumptions does this rely on that could be wrong?
- What's the strongest argument AGAINST this recommendation?
- Are there edge cases or failure modes not addressed?
- If you had to improve one thing about this recommendation, what would it be?
```

## Contradiction Resolution Template

Use this when you hit Pattern 6 (Confident Disagreement), where two models take opposing positions with strong reasoning on both sides.

```
Two analyses confidently disagree on the following point:

POSITION A: [state clearly]
REASONING A: [summarize]

POSITION B: [state clearly]
REASONING B: [summarize]

YOUR TASK:
- Evaluate both positions on their merits
- Identify what evidence would settle the disagreement
- If you must choose one, which position is better supported and why?
- Is there a third option that resolves the contradiction?
```

## Filled Example: Caching Strategy Reconciliation

```
I have collected independent analyses from 3 AI models on the following question:

ORIGINAL QUESTION: Should we add a caching layer for our product catalog API? Current p95 latency is 820ms, target is under 200ms. Stack is Node.js, PostgreSQL, 50k products, ~2,000 requests/minute.

MODEL A RESPONSE (SUMMARY):
- Recommends Redis with a 5-minute TTL and cache-aside pattern
- Estimates p95 will drop to ~40ms for cache hits
- Suggests invalidating on product update via a pub/sub channel
- Warns about cold-start latency after deploys

MODEL B RESPONSE (SUMMARY):
- Recommends a two-tier approach: in-process LRU cache (node-cache) for hot items, Redis for the full catalog
- Estimates p95 under 15ms for top 500 products, ~50ms for the rest
- Suggests write-through invalidation tied to the ORM's save hooks
- Warns about memory pressure if in-process cache is too large

MODEL C RESPONSE (SUMMARY):
- Recommends optimizing PostgreSQL first (missing index on category_id, unoptimized JOIN on variants table) before adding a cache
- Estimates query optimization alone could bring p95 to ~150ms
- If caching is still needed after optimization, recommends Redis with 10-minute TTL
- Warns that caching without fixing the query masks a debt that will compound

YOUR TASK:
1. Identify where the models AGREE (high-confidence signals)
2. Identify where the models DISAGREE (needs investigation)
3. Flag any claims that appear unsupported or potentially hallucinated
4. Synthesize the strongest elements from each response into a single, unified recommendation
5. Note any tradeoffs or caveats the final recommendation should acknowledge
6. Rate your confidence in the synthesis: HIGH / MEDIUM / LOW
```

**What good reconciliation output looks like for this example:**

- All three models agree caching will help; Redis is the consensus technology choice.
- Model C's point about query optimization is complementary, not contradictory. Fix the queries first, then add caching. This is a sequencing insight, not a disagreement.
- Model B's two-tier approach adds complexity that may not be justified for 2,000 req/min. Worth flagging as a "revisit at scale" option rather than a launch requirement.
- Model A's cold-start warning and Model C's masking-debt warning are both valid caveats that belong in the final recommendation.
- The TTL disagreement (5 min vs 10 min) is minor and testable. Start with 5 minutes and adjust based on cache-hit ratio.

## When to Use Each Template

| Situation | Template |
|---|---|
| Standard council session with 2-4 responses | Primary Reconciliation |
| High-stakes decision (architecture, security, data model) | Primary Reconciliation, then Triple-Audit |
| Two models directly contradict each other | Contradiction Resolution |
| All models agree but you want to stress-test | Triple-Audit alone |
