# Reconciliation Patterns: Reading the Room

After you collect responses from your panel, you'll notice patterns in how models agree or disagree. Recognizing these patterns quickly tells you what to do next.

After 500+ sessions, these six patterns cover nearly every scenario. Learn to spot them fast and you'll cut your reconciliation time in half.


## Pattern 1: Strong Consensus

**What it looks like:** All models independently reach the same conclusion with similar reasoning.

**Confidence level:** High.

**What to do:** Proceed. Pick the best-articulated version, verify the shared reasoning is sound, and lock it in.

**Watch out for:** False consensus. All models were trained on overlapping data, so they can all be wrong in the same way. If the stakes are high, verify with a contrarian prompt before committing.

**Example scenario:**
You ask 4 models whether to use PostgreSQL or MongoDB for a transactional e-commerce system. All four recommend PostgreSQL with ACID compliance as the primary reason. This is strong consensus. Proceed.


## Pattern 2: Majority with Outlier

**What it looks like:** 3 models agree, 1 takes a different position.

**Confidence level:** Medium-high (but investigate the outlier).

**What to do:** Don't dismiss the outlier. Read its reasoning carefully. Often the outlier caught something the majority missed. If the outlier's reasoning is weak, proceed with the majority. If it's strong, run a focused follow-up.

**Watch out for:** The temptation to skip the outlier because "majority rules." That defeats the purpose of using multiple models.

**Example scenario:**
You ask about caching strategy. Three models recommend Redis. Grok suggests you don't need a cache at all and should optimize the database queries first. The outlier might be right: over-caching is a real anti-pattern. Investigate before proceeding.


## Pattern 3: Split Decision

**What it looks like:** A 2v2 split, or fragmented opinions with no clear majority.

**Confidence level:** Low. This is a genuinely hard problem.

**What to do:** Don't force a decision. Reframe the question with more constraints. Ask each "side" to steel-man the opposing view. Often the split reveals a missing requirement or an implicit assumption that needs to be made explicit.

**Watch out for:** Picking a side just to move forward. A split decision is a signal, not a problem to brute-force.

**Example scenario:**
Two models recommend microservices, two recommend a modular monolith. The split usually means "it depends on factors you haven't specified." Add constraints (team size, deployment timeline, scale expectations) and re-run.


## Pattern 4: Universal Uncertainty

**What it looks like:** All models hedge, use qualifying language, and give conditional answers.

**Confidence level:** Very low. But this is valuable information.

**What to do:** Recognize that you've hit a genuinely ambiguous problem. No amount of re-prompting will create certainty that doesn't exist. Make a documented decision, note the uncertainty, and plan for reversibility.

**Watch out for:** Mistaking honest uncertainty for model failure. When all four models say "it depends," they're telling you something real.

**Example scenario:**
You ask about the long-term viability of a new framework. All models say "it depends on adoption trends" and "too early to tell." This is honest uncertainty. Pick the option with the lowest switching cost and move forward.


## Pattern 5: Complementary Fragments

**What it looks like:** Each model contributes a unique, valuable piece, but none has the complete picture.

**Confidence level:** High (once assembled).

**What to do:** This is the council method at its best. Synthesize the fragments. Claude catches the architecture concern, GPT finds the creative solution, Gemini provides the data backing, Grok identifies the edge case. Your job is assembly.

**Watch out for:** Forcing fragments together when they actually conflict. Make sure the pieces are genuinely complementary before combining them.

**Example scenario:**
Code review of authentication middleware. Claude flags coupling issues. GPT spots a token refresh edge case. Gemini identifies a performance bottleneck. Grok finds an unusual attack vector. None of these overlap. Together, they form a comprehensive review no single model would have produced.


## Pattern 6: Confident Disagreement

**What it looks like:** Models are certain but contradictory. No hedging, no uncertainty, just flat-out disagreement.

**Confidence level:** Dangerous. This is the highest-risk pattern.

**What to do:** Do NOT pick a side based on which model you trust more. This pattern means at least one model is confidently wrong. Dig into primary sources. Test the claims independently. If it's a code question, write the test. If it's an architecture question, prototype both approaches.

**Watch out for:** Splitting the difference. When two models confidently contradict each other, the truth is usually one or the other, not somewhere in between.

**Example scenario:**
One model insists a certain API supports streaming natively. Another model insists it doesn't. Both are confident. One of them is hallucinating or working from outdated information. Check the actual documentation.


## Red Flags to Watch For

These failure modes can corrupt your results even when the patterns above look clean.

- **Groupthink cascade:** Models converge too quickly on a trendy answer. Check if they're all pulling from the same popular blog post or Stack Overflow thread. Popularity is not correctness.

- **Hallucination echo:** One model's hallucinated claim gets validated in reconciliation because it sounds plausible. Always verify specific technical claims against primary sources.

- **Authority bias:** Defaulting to Claude's or GPT's answer because they're "the big models." The method works because you weigh reasoning, not reputation. A smaller model with better reasoning wins.

- **False consensus:** All models agree, but they're all wrong in the same way. This is common with training data biases. Use contrarian prompting to stress-test unanimous answers.

- **Complexity creep:** The reconciled answer is more complex than any individual answer. Simpler is usually better. The council should reduce complexity, not add it. If your synthesis is longer than the longest input, something went wrong.


## Quick Decision Tree

Use this as a fast reference during reconciliation:

- **All agree with strong reasoning** → Lock it (Pattern 1)
- **Mostly agree, one outlier** → Investigate the outlier, then decide (Pattern 2)
- **Split or fragmented** → Add constraints, re-run (Pattern 3)
- **All uncertain** → Make a reversible decision, document it (Pattern 4)
- **Each contributes a piece** → Assemble the pieces (Pattern 5)
- **Confident contradiction** → Verify independently, trust nothing (Pattern 6)

When in doubt, ask yourself: "Am I picking this answer because the reasoning is strongest, or because it's the most comfortable?" If it's comfort, slow down.

---

**Next:** [Quality Rules](04_quality_rules.md) | **Back to:** [Overview](00_overview.md) | **Home:** [README](../README.md)
