# Quality Rules: The Non-Negotiables

These rules emerged from 500+ council sessions and 390+ documented rules. Most of those rules are project-specific. The ones below are universal. They apply to every council session, every project, every team size.


## Rule 1: 150% Is the Acceptance Floor, 200% Is the Design Target

"Good enough" is not good enough. If you're going to run a council session, the output should be significantly better than what any single model would produce alone.

- **150%** means the council output is clearly, measurably better than the best single-model response. This is the minimum bar for the council to justify its overhead.
- **200%** means the council output does things no single response could have done. Novel synthesis, caught blind spots, resolved contradictions, produced something greater than the sum of its parts.
- If your council outputs routinely land at **100%** (same quality as a single model), your prompts need work or you're using the council for tasks that don't benefit from it (see [When to Use the Council](05_when_to_use.md)).
- **How to measure:** compare the final council output against the best individual response. Would you ship the individual response as-is? If yes, the council didn't add enough value.


## Rule 2: Triple-Audit Before Locking

Every major decision gets verified three ways before it's locked:

1. **Independent analysis** (the original council responses)
2. **Reconciliation** (synthesized output)
3. **Adversarial review** (at least one model challenges the synthesis)

Skipping the third step is the most common mistake. It's also where the most value hides.

A locked artifact means: "This was independently analyzed, reconciled, and challenged. The orchestrator reviewed all three rounds and approved."

Not everything needs triple-audit. Use it for architecture decisions, security-critical code, production deployments, and anything that's expensive to change later.


## Rule 3: No Single Model Owns the Truth

The moment you start treating one model as "the smart one," the council stops working.

Every model has blind spots. Claude over-engineers. GPT gets confidently creative with facts. Gemini defaults to Google solutions. Grok can be contrarian for the sake of it.

The human orchestrator is the final authority. Not Claude. Not GPT. You.

If you find yourself always siding with the same model, you're not running a council. You're using one model with extra steps.


## Rule 4: Cross-Model Consensus Is Signal, Not Proof

When all models agree, it increases confidence. It does not guarantee correctness.

Models share training data. They share biases. They can all be wrong in the same way.

- **Consensus on factual claims:** verify against primary sources.
- **Consensus on architecture:** stress-test with edge cases and scale scenarios.
- **Consensus on "best practices":** ask whether the best practice applies to your specific context.

The value of consensus is that it narrows the search space. You still have to verify.


## Rule 5: Document Every Decision with Rationale

Every council session produces a log. Every log includes: the question, the panel, a summary of each response, the pattern observed, the final decision, and the rationale.

Rationale means: why this decision, why not the alternatives, what dissenting views were considered and why they were set aside.

This documentation is not bureaucracy. It's insurance. When something breaks six months later, the decision log tells you what assumptions were in play.

Use the [Session Log Template](../templates/session_log_template.md).


## Rule 6: Export Before Context Gets Heavy

AI models have context windows. As conversations get longer, quality degrades. Models forget earlier context. Reasoning gets circular.

Export your artifacts (decisions, code, documentation) to files outside the session before the context window fills up.

**Rule of thumb:** if you've been in a session for more than 60 minutes or more than 30 back-and-forth exchanges, export everything valuable and start a fresh session.

Losing work to a crashed or degraded session is entirely preventable. Export early, export often.

For critical work, copy key decisions to a separate document after every major milestone within the session.


## Rule 7: Name Things Once, Name Them Right

When the council produces terminology, variable names, or component names, lock them immediately.

Inconsistent naming across files (because different models used different terms) is the number one integration bug in council-produced code.

The reconciliation step should explicitly standardize naming. If Claude calls it "AuthService" and GPT calls it "AuthenticationManager," pick one in reconciliation and enforce it everywhere.


## Rule 8: Never Ship Unreconciled Output

Raw model output goes through reconciliation before it touches production. No exceptions.

"But this response looks really good" is not a reason to skip reconciliation. Single-model outputs have passed your first impression filter, not a council review.

The reconciliation step is where quality happens. The parallel analysis step is where coverage happens. You need both.


## Building Your Own Rule Library

Start with these 8 rules. They cover the fundamentals.

After every council session, ask: "What went wrong? What went unexpectedly right?"

- If the same issue comes up three times, it becomes a rule.
- Write rules as imperatives, not suggestions: "Export before context gets heavy" not "Consider exporting when context might be getting long."
- Organize rules by category: process rules, quality rules, model-specific rules, project-specific rules.
- The 390+ rules in the full knowledge base started from a list of 10. They grew organically over 500+ sessions. Yours will too.


## The Meta-Rule

Rules exist to prevent repeated mistakes and encode hard-won knowledge. They are not compliance checkboxes.

If a rule doesn't make sense for a specific situation, document why you're breaking it and move on. Blind rule-following is its own failure mode.

The best rule library is one that evolves. Review your rules quarterly. Kill the ones that no longer apply. Sharpen the ones that keep saving you.


---

**Next:** [When to Use the Council](05_when_to_use.md) | **Back to:** [Overview](00_overview.md) | **Home:** [README](../README.md)
