# Model Roles: Casting Your Panel

You are building a panel of four AI models. Each one brings different strengths, different blind spots, and different instincts. Your job is to know what each model does well, where it stumbles, and how to cast it for maximum impact.

This is not about finding the "best" model. It is about assembling a team where the members compensate for each other's weaknesses.

---

## Claude (Anthropic)

### Strengths

- **Architecture and system design.** Give Claude a complex system to reason about, and it will map out components, boundaries, and data flows with precision.
- **Structured reasoning.** Excels at step-by-step analysis, especially when tradeoffs are involved. Will lay out pros, cons, and second-order consequences without being asked.
- **Long-context analysis.** Handles large codebases and lengthy documents without losing the thread.
- **Code quality and consistency.** Produces clean, well-organized code that follows conventions. Strong at maintaining consistency across files.
- **Documentation.** Generates clear, thorough documentation that actually matches the code.
- **Nuanced tradeoff analysis.** When there is no single right answer, Claude maps the decision space instead of forcing a pick.

### Watch-Outs

- Can be **overly cautious**. Will sometimes add caveats and disclaimers when you need a direct answer.
- Tends to **over-engineer**. May suggest abstractions or patterns that add complexity without enough payoff.
- **Hedges when pressed.** If you need a definitive recommendation, you may need to push for one explicitly.

### Best Cast As

Lead architect, reconciliation engine, final QA reviewer, documentation lead.

---

## GPT-4 (OpenAI)

### Strengths

- **Creative problem-solving.** Generates approaches and framings you would not have considered. Thinks laterally.
- **Broad knowledge base.** Strong general knowledge across domains, making it useful for cross-disciplinary tasks.
- **Alternative approaches.** Reliably surfaces "have you considered..." options that expand your solution space.
- **Rapid prototyping.** Produces working code quickly, even if it is not always production-ready.
- **Concept explanation.** Excellent at breaking down complex ideas into clear, accessible language.

### Watch-Outs

- Can be **confidently wrong**. Will present incorrect information with the same tone as correct information. Always verify.
- Sometimes **prioritizes cleverness over simplicity**. May reach for an elegant solution when a straightforward one would serve better.
- **Drifts from constraints.** If you gave it specific requirements, check that the output actually satisfies all of them.

### Best Cast As

Creative brainstormer, alternative approach generator, rapid prototype builder, knowledge base for cross-domain questions.

---

## Gemini (Google)

### Strengths

- **Data analysis patterns.** Strong at identifying patterns in data, structuring analysis pipelines, and working with tabular information.
- **Cross-referencing.** Good at pulling together information from multiple sources and spotting connections.
- **Speed.** Fast response times, especially useful during iterative exploration.
- **Google ecosystem integration.** If your stack touches Google Cloud, BigQuery, Firebase, or similar tools, Gemini has deep familiarity.
- **Technical research depth.** Thorough at surveying a technical landscape and summarizing the state of the art.
- **Structured comparisons.** Produces clean comparison matrices and feature-by-feature breakdowns.

### Watch-Outs

- Can be **verbose**. May give you three paragraphs where three sentences would do. Be ready to ask for concise output.
- Sometimes **surfaces quantity over quality**. Will list ten options when the top three would have been more useful.
- **Defaults to Google-ecosystem solutions.** When multiple tools could solve a problem, Gemini may lean toward the Google option without evaluating alternatives fairly.

### Best Cast As

Research lead, data analysis specialist, comparison matrix builder, technical deep-diver.

---

## Grok (xAI)

### Strengths

- **Unfiltered analysis.** Less likely to soften bad news or avoid uncomfortable conclusions.
- **Contrarian perspectives.** Actively challenges assumptions and conventional wisdom. Will tell you why your plan might fail.
- **Edge case hunting.** Strong at identifying scenarios others overlook, including adversarial and failure modes.
- **Real-time data access.** Can pull in current information that other models may lack.
- **Catches things others miss.** Because it reasons differently, it often flags issues that the other three models agree to ignore.

### Watch-Outs

- Can be **too contrarian**. Sometimes challenges assumptions that are actually well-founded, creating noise.
- May **prioritize being different over being right**. Evaluate its objections on merit, not novelty.
- **Less polished output.** Formatting and structure may need cleanup before sharing with stakeholders.

### Best Cast As

Devil's advocate, edge case hunter, assumption challenger, unconventional angle provider.

---

## Role Assignment Matrix

Use this table as a starting point. Adjust based on your experience with each model over time.

| Task Type | Recommended Lead | Full Panel? | Notes |
|---|---|---|---|
| Architecture decisions | Claude | Yes | All four weigh in; Claude synthesizes |
| Code review | Claude | Optional | Add GPT for alternative patterns, Grok for edge cases |
| Debugging | GPT | Optional | GPT for creative hypotheses; Claude for systematic analysis |
| Research | Gemini | Yes | Gemini leads; others validate and add angles |
| Creative/brainstorming | GPT | Yes | GPT leads; Grok adds contrarian takes |
| Documentation | Claude | No | Claude solo is usually sufficient |
| Security review | Grok | Yes | Grok hunts adversarial cases; Claude checks systematically |
| Performance optimization | Gemini | Optional | Gemini for data-driven analysis; Claude for architectural impact |

---

## Overlap Is a Feature

If you are coming from an efficiency mindset, running the same task through four models feels wasteful. Resist that instinct. Redundancy is the entire point.

**When multiple models independently reach the same conclusion, your confidence should go up.** Three out of four models recommending the same database architecture is a stronger signal than any single model's recommendation alone.

**When they disagree, you have found something worth investigating.** Disagreement is not a problem to resolve quickly. It is a flag that says: "This decision has hidden complexity. Slow down and look closer."

You are not buying efficiency. You are buying coverage. The cost of running four models is trivial compared to the cost of a bad architectural decision that survives for months before someone catches it.

Think of it like code review. You do not skip review because "the author already tested it." You add reviewers because different people catch different things. Same principle, different reviewers.

---

## Rotating the Lead

Every council session needs a lead: the model whose output you start from when reconciling. The lead frames the discussion. Choose it based on the task.

### Claude as Default Reconciler

Claude is your default choice for the reconciliation step. It excels at synthesis, holding multiple perspectives in context, and producing structured output that integrates competing viewpoints. For most tasks, start here.

### When GPT Should Lead

Put GPT in the lead seat for **creative and generative tasks**. Naming things, brainstorming features, exploring solution spaces, writing user-facing copy. GPT's lateral thinking sets a wider frame that the other models can then refine and pressure-test.

### When Gemini Should Lead

Gemini should lead on **data-heavy and research tasks**. Evaluating technology options, surveying a competitive landscape, analyzing performance benchmarks, building comparison matrices. Gemini's structured, thorough approach gives you a solid foundation that others can challenge.

### When Grok Should Lead

Give Grok the lead on **adversarial and risk-focused tasks**. Threat modeling, security review, "what could go wrong" analysis, pre-mortem exercises. Starting from Grok's contrarian perspective forces the other models to defend their assumptions instead of reinforcing them.

### The Pattern

1. Pick the lead based on task type.
2. Send the same prompt to all four models.
3. Read the lead's response first to set your frame.
4. Read the other three for additions, corrections, and challenges.
5. Reconcile using the lead (or Claude, if the lead is not strong at synthesis).

You will develop your own instincts for this. The matrix above is a starting point, not a rulebook.

---

**Next:** [Session Workflow](02_session_workflow.md) | **Back to:** [Overview](00_overview.md) | **Home:** [README](../README.md)
