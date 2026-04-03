# Session Workflow: Running a Council Session

A council session has eight steps. Most take 5-10 minutes each. A full session runs 45-90 minutes depending on the complexity of the question. With practice, you develop a rhythm, and the process starts to feel less like a procedure and more like a conversation you're conducting.

This guide walks through every step. Follow it linearly the first few times. Once you internalize the flow, you will naturally learn where to compress and where to linger.

---

## Step 1: Define the Question

The quality of your council session depends entirely on the quality of your prompt. A vague question produces vague answers from every model, and no amount of reconciliation fixes that.

Write the question once, then send it to all models unchanged. One prompt, multiple recipients, zero variation.

Be specific:

- **Good:** "Should we use WebSockets or SSE for real-time updates in our trading dashboard, given a React frontend, a Node.js backend, and the need to support 10k concurrent connections?"
- **Bad:** "What's best for real-time?"

Include relevant context in every prompt:

- Your current stack and infrastructure
- Hard constraints (budget, timeline, compliance)
- What you have already tried and why it did not work
- What outcome you need (a recommendation, a comparison, a code sample)

See the [Council Prompt Template](../templates/council_prompt_template.md) for a ready-to-use structure.

---

## Step 2: Select Your Panel

Not every question needs four models. Match your panel size to the stakes.

- **2 models:** Fast comparison. Good for focused technical questions with a clear correct answer.
- **3 models:** Balanced coverage. The sweet spot for most tasks. Gives you a tiebreaker if two models disagree.
- **4 models:** Maximum coverage. Use this for architecture decisions, critical path choices, and anything that will be expensive to reverse.

Do not always use the same panel. Rotate based on the task domain. A code generation task calls for a different lineup than a strategic planning question. Reference [Model Roles](01_model_roles.md) for guidance on which models excel where.

---

## Step 3: Distribute to Models

Same prompt, separate sessions. This is critical.

- Use incognito or private windows, or separate browser profiles, for each model.
- Never paste one model's output into another model's session during this phase.
- If you are using API access, make separate API calls with no shared context.
- Do not mention other models or their responses in your prompts.

The independence of responses is what makes the method work. The moment one model sees another's output, you lose the benefit of independent reasoning. You are not looking for agreement; you are looking for convergence that emerges naturally from isolated analysis.

---

## Step 4: Collect Raw Outputs

Read each response fully before comparing. Do not skim. Each model structures its thinking differently, and surface-level scanning will cause you to miss important nuances buried in the middle of a response.

- Copy each response into your session log. See the [Session Log Template](../templates/session_log_template.md) for the format.
- Note your initial impression of each response: what stood out, what felt weak, what surprised you.
- Resist the urge to start comparing yet. Just read and record.

This discipline matters. Your first impressions, captured before comparison, become valuable data during triage.

---

## Step 5: Human Triage

This is where the orchestrator earns their keep.

Read all responses side by side. You are looking for four things:

### Convergence

Where do three or more models agree? This is a high-confidence signal. If independent models reach the same conclusion through different reasoning paths, you can trust that conclusion more than any single model's output.

### Divergence

Where do models disagree? These are the areas that need investigation. Divergence is not a problem; it is information. It tells you the question has genuine complexity that a single model might gloss over.

### Hallucinations

Flag claims that seem specific but lack sourcing. Watch for invented API names, fabricated statistics, nonexistent libraries, and citations to papers that do not exist. Each model has its own hallucination patterns.

### Red Flags

- Overly confident answers with no caveats
- Circular reasoning (restating the question as the answer)
- Ignoring constraints you explicitly listed in the prompt
- Recommending deprecated or insecure approaches

Score each response on four dimensions:

| Dimension      | What to look for                                    |
|----------------|-----------------------------------------------------|
| Accuracy       | Are the facts correct? Are claims verifiable?       |
| Completeness   | Did it address every part of your question?         |
| Actionability  | Can you act on this advice without further research?|
| Creativity     | Did it surface options or angles you had not considered? |

Use the patterns from [Reconciliation Patterns](03_reconciliation_patterns.md) to classify what you are seeing.

---

## Step 6: Reconciliation Round

Pick your reconciliation lead. For most synthesis tasks, Claude handles this well, but choose based on the domain.

Feed the reconciler a structured prompt with the key findings from all models. Do not dump raw outputs. Curate the input:

> "Model A recommended WebSockets because of bidirectional communication needs. Model B recommended SSE because of simpler infrastructure and sufficient functionality for our read-heavy use case. Model C agreed with Model A but added a caveat about connection limits behind our load balancer."

Ask the reconciler to:

- Synthesize the strongest elements from each response
- Flag contradictions that remain unresolved
- Provide a unified recommendation with clear reasoning
- Note any risks or tradeoffs the final recommendation carries

See the [Reconciliation Prompt Template](../templates/reconciliation_prompt_template.md) for the full prompt structure.

---

## Step 7: Triple-Audit Verification

Take the reconciled output and feed it back to one or two of the original panel models. Use a prompt like this:

> "Here is a synthesized recommendation for [topic]. Challenge it. What did we miss? What assumptions are wrong? What failure modes should we worry about?"

This step catches two failure modes:

1. **Groupthink:** The reconciler may have smoothed over a legitimate disagreement instead of preserving it.
2. **Reconciliation bias:** The reconciler may have favored the most articulate response rather than the most correct one.

If the auditing models raise valid concerns, loop back to Step 6 with the new information. This is not a failure; it is the process working as designed.

This step is what separates good council sessions from great ones. Do not skip it.

---

## Step 8: Lock and Export

The final output gets a version tag in your session log. A locked artifact means: this was council-verified, triple-audited, and the orchestrator approved it.

- Export the decision, the rationale, and any dissenting views to your project documentation.
- Do this BEFORE the context window gets heavy. If you wait too long, you risk losing nuance as models begin to lose earlier context.
- Record which models participated, what the key disagreements were, and how they were resolved.
- Tag the artifact with a confidence level (high, medium, low) based on the degree of convergence.

Reference [Quality Rules](04_quality_rules.md) for the full locking criteria and artifact standards.

---

## Session Timing Guide

Use these benchmarks to plan your sessions:

| Session Type    | Panel Size | Complexity | Time Estimate |
|-----------------|------------|------------|---------------|
| Quick council   | 2 models   | Simple, focused question | 20-30 minutes |
| Standard council| 3 models   | Moderate complexity      | 45-60 minutes |
| Deep council    | 4 models   | Architecture, critical path | 60-90 minutes |

If a session goes past 90 minutes, stop. Split the question into smaller, more focused pieces and run separate council sessions for each. Long sessions produce diminishing returns as context windows fill up and your own attention drifts.

---

## When to Cut a Session Short

Not every session needs all eight steps. Cut early when:

- **Strong consensus at Step 5.** If all models agree on the approach, the reasoning is sound, and no red flags appear, skip directly to Step 8. The triple-audit adds less value when there is nothing to challenge.
- **The question needs reframing.** If the responses reveal that you asked the wrong question, stop. Rewrite the prompt and start over. A better question in a new session beats a forced answer in a broken one.
- **Context windows are getting long.** If you have been going back and forth and responses start losing coherence, export what you have and start a fresh session. Partial progress, properly exported, is never wasted.

---

## Common Mistakes

These are the errors that undermine council sessions most often. Learn to recognize them.

**Letting one model see another's output.** This kills independence and turns your council into an echo chamber. Keep sessions completely isolated until you reach the reconciliation step.

**Skipping the triple-audit (Step 7) to save time.** This is where the real value is. The reconciliation step produces a polished answer; the audit step tests whether that answer holds up under pressure. Skipping it is like writing code without running the tests.

**Using the same panel every time.** Different questions benefit from different model combinations. A model that excels at code review may not be the best choice for market analysis. Rotate based on the task type.

**Not exporting before context gets heavy.** Context windows are finite. If you wait too long to lock and export, you lose work. Export early, export often, and always export before starting extended follow-up conversations.

**Treating model agreement as proof of correctness.** Convergence is a signal, not a guarantee. All models can be wrong about the same thing, especially on topics where training data contains widespread misconceptions. Your judgment as orchestrator is the final check.

---

**Next:** [Reconciliation Patterns](03_reconciliation_patterns.md) | **Back to:** [Overview](00_overview.md) | **Home:** [README](../README.md)
