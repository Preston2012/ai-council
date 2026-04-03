# When to Use the Council (and When Not To)

The council method has overhead. You're running multiple sessions, reading multiple responses, reconciling, auditing. That overhead pays for itself on complex, high-stakes tasks. It's wasted on simple ones.

This guide helps you decide when to spin up a full council and when a single model gets the job done.

## The Three Tiers

### Always Council

These tasks benefit from multi-model analysis every time. The cost of getting them wrong is high, and single-model blind spots are most dangerous here.

| Task | Why Council | Recommended Panel Size |
|------|-------------|----------------------|
| Architecture decisions | Wrong architecture is expensive to fix. Multiple perspectives catch assumptions. | 4 models |
| Security reviews | Each model catches different vulnerability classes. Coverage matters. | 3-4 models |
| Production core logic | The code that runs your business needs maximum scrutiny. | 3-4 models |
| Technology selection | Lock-in risk is real. Models have different ecosystem biases that balance out. | 4 models |
| API design | Once published, APIs are hard to change. Get it right the first time. | 3 models |
| Data model design | Schema decisions ripple through everything. Multiple models catch normalization issues others miss. | 3-4 models |

### Council Recommended

These tasks benefit from council analysis most of the time. Use your judgment on complexity. If it feels straightforward, a single model might be fine. If there's any ambiguity, run the council.

| Task | Why Council Helps | Recommended Panel Size |
|------|-------------------|----------------------|
| Complex debugging | Each model diagnoses differently. One finds the root cause others miss. | 2-3 models |
| Research synthesis | Models pull from different knowledge bases. Combined coverage is broader. | 3-4 models |
| Code review (critical paths) | Complementary fragments pattern surfaces here constantly. | 3 models |
| Performance optimization | Models suggest different optimization strategies. Testing multiple approaches is faster than guessing. | 2-3 models |
| Error handling design | Edge cases are each model's specialty. Collective edge case coverage is much stronger. | 3 models |
| Migration planning | Risk assessment benefits from multiple perspectives. | 3 models |

### Single Model Is Fine

These tasks don't benefit enough from council analysis to justify the overhead. Use one model, apply human QA, and move on.

| Task | Why Single Model Works | Notes |
|------|----------------------|-------|
| Quick factual questions | The answer is verifiable. If the model is wrong, you'll know fast. | Verify critical facts regardless |
| Boilerplate generation | Standard patterns don't need multiple opinions. | Review output, don't blindly ship |
| Documentation formatting | Style and structure, not substance. One model handles this. | |
| Simple CRUD operations | Well-understood patterns with low ambiguity. | |
| Commit messages and PR descriptions | Quick, low-stakes text. | |
| Email and communication drafts | Subjective output where one perspective is sufficient. | Human QA is still essential |
| Configuration file generation | Deterministic output with known correct answers. | |

## The Decision Flowchart

Ask these questions in order:

1. **Is this reversible?** If you can easily undo a bad decision, single model is fine. If reversal is expensive (architecture, public API, data schema), use the council.

2. **Is this ambiguous?** If reasonable engineers would disagree on the approach, use the council. If there's a clear best practice, single model is fine.

3. **What's the blast radius?** If a mistake affects one file, single model. If it affects the entire system, council.

4. **Is this novel?** If you've solved this exact problem before, single model. If it's new territory, the council catches things you don't know to look for.

5. **Will you sleep well?** If a wrong answer here would keep you up at night, run the council.

## Cost-Benefit Analysis

Running a council session costs time. Here's what that looks like in practice:

| Session Type | Time Investment | Quality Gain | Break-Even Point |
|-------------|----------------|--------------|------------------|
| 2-model quick comparison | 20-30 minutes | 20-40% better coverage | Tasks that take 2+ hours to fix if wrong |
| 3-model standard council | 45-60 minutes | 40-70% better coverage | Tasks that take 4+ hours to fix if wrong |
| 4-model deep council | 60-90 minutes | 60-100%+ better coverage | Architecture and decisions that affect weeks of work |

The math is simple: if the council prevents one bad architecture decision per week, it saves days of rework. Over 500+ sessions, the methodology has prevented hundreds of hours of backtracking.

## Scaling Your Panel

Not every council needs four models. Here's how to scale:

**2 Models: Speed Run**
- Best for: quick comparisons, A/B decisions, sanity checks
- Pair suggestion: Claude + GPT (broadest coverage for most tasks)
- When to use: time pressure, moderate stakes, well-scoped question

**3 Models: The Sweet Spot**
- Best for: most development tasks, code review, research
- Panel suggestion: Claude + GPT + Gemini (covers architecture, creativity, and data)
- When to use: default for any task that warrants a council at all

**4 Models: Full Council**
- Best for: architecture decisions, security reviews, critical production changes
- Panel: Claude + GPT + Gemini + Grok (maximum coverage and adversarial pressure)
- When to use: high-stakes, hard-to-reverse decisions

## When to Skip Council Even for Important Tasks

Sometimes the overhead isn't justified even for important work:

- **Time-critical production issues**: Fix it with one model, document the fix, then council-review the fix afterward when the pressure is off.
- **Well-understood domain**: If you've council-reviewed the same pattern 10 times and the answer is always the same, you've graduated from needing a council for that pattern.
- **Incremental changes**: If you're adding a field to an existing, council-approved schema, you don't need a new council session. Apply the established pattern.
- **Prototyping and exploration**: When you're just trying things to see what works, single model is faster. Council the final approach, not every experiment.

## Real Results

This decision framework, applied across 500+ sessions, produced:

- **6 production applications** shipped and running
- **100,000+ lines of code** across multiple tech stacks
- **390+ quality rules** refined through real use
- **Zero major architecture reversals** in council-reviewed decisions

The council method isn't about using it everywhere. It's about knowing when the overhead buys you something single-model workflows can't: coverage, confidence, and correctness on the decisions that matter most.

See the results in action:
- [Baseline Showcase](https://github.com/Preston2012/baseline-showcase)
- [Trading Toolkit](https://github.com/Preston2012/trading-toolkit)
- [Portfolio](https://baseline.marketing/built)

---
**Back to:** [Overview](00_overview.md) | **Home:** [README](../README.md)
