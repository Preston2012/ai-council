# AI Council Method

**A practitioner's framework for multi-model AI orchestration.**

One model gives you one perspective. You have no way to know what it missed until production tells you. The AI Council Method fixes that by running multiple AI models in parallel on the same task, then using human judgment to reconcile the strongest elements into a single, verified output.

This is not an automated pipeline. This is not a wrapper or an SDK. This is a documented methodology for how one engineer orchestrates Claude, GPT, Gemini, and Grok to produce work that no single model can match alone.

---

### Results

| Metric | Count |
|--------|-------|
| Application projects produced | 6 (1 published, 1 closed testing, 1 prototype, 3 in development/design) |
| Lines of code produced | 100,000+ |
| Quality rules documented | 390+ |
| Build sessions completed | 500+ |

---

## What Is This?

This is a methodology repository. It contains the complete, documented process for running multi-model AI council sessions: sending the same task to 2-4 AI models independently, collecting their outputs without cross-contamination, and reconciling the best elements into a final deliverable under human supervision.

The methodology was developed across 500+ real build sessions producing production software. Every pattern, rule, and template in this repo comes from practical experience, not theory.

**The core insight:** When multiple models independently analyze the same problem, their areas of agreement become high-confidence signals. Their areas of disagreement become investigation targets. And their unique contributions combine into outputs that are consistently stronger than any single model's response.

## Quick Start

New to the council method? Read these three files in order:

1. **[Overview](methodology/00_overview.md)** - What the method is, why it works, and the core terminology
2. **[Session Workflow](methodology/02_session_workflow.md)** - Step-by-step instructions for running your first council session
3. **[When to Use the Council](methodology/05_when_to_use.md)** - Deciding when the overhead is worth it

You can run your first council session after reading those three files. The rest of the repo deepens your understanding and provides ready-to-use tools.

## Repository Structure

```
ai-council/
├── methodology/
│   ├── 00_overview.md              # What, why, and how (start here)
│   ├── 01_model_roles.md           # Strengths and casting guide for each model
│   ├── 02_session_workflow.md      # 8-step workflow for running a session
│   ├── 03_reconciliation_patterns.md  # 6 patterns for reading model agreement/disagreement
│   ├── 04_quality_rules.md         # The non-negotiable rules (150%/200%, triple-audit, etc.)
│   └── 05_when_to_use.md           # Decision framework: council vs. single model
├── examples/
│   ├── architecture_decision.md    # Full council session: data pipeline architecture
│   ├── code_review_session.md      # Parallel code review catching 4x the issues
│   └── research_synthesis.md       # Framework evaluation with weighted decision matrix
├── templates/
│   ├── council_prompt_template.md      # Copy-paste prompt for council sessions
│   ├── reconciliation_prompt_template.md  # Synthesis and verification prompts
│   └── session_log_template.md         # Session documentation template
├── LICENSE
└── README.md
```

## Methodology

The methodology folder contains six files that cover the complete process from concept to execution.

### [Overview](methodology/00_overview.md)
The conceptual foundation. Explains why single-model workflows hit a ceiling, defines the core terminology (council session, orchestrator, panel, reconciliation), and maps out the reading order for the rest of the methodology.

### [Model Roles](methodology/01_model_roles.md)
A casting guide for Claude, GPT-4, Gemini, and Grok. Each model gets a profile covering strengths, watch-outs, and ideal use cases. Includes a role assignment matrix and guidance on rotating the lead role based on task type.

### [Session Workflow](methodology/02_session_workflow.md)
The procedural core. Eight steps from defining the question through locking the final artifact. Covers prompt distribution, human triage, reconciliation, triple-audit verification, and context management. This is the file you reference during every session.

### [Reconciliation Patterns](methodology/03_reconciliation_patterns.md)
Six named patterns for how models agree and disagree: Strong Consensus, Majority with Outlier, Split Decision, Universal Uncertainty, Complementary Fragments, and Confident Disagreement. Each pattern includes recognition criteria, recommended actions, and real scenarios. Also covers red flags like groupthink cascading and hallucination echo.

### [Quality Rules](methodology/04_quality_rules.md)
Eight universal rules extracted from 390+ documented rules. Covers the 150%/200% quality bar, triple-audit protocol, consensus verification, decision documentation, and context management. Includes guidance on building your own rule library.

### [When to Use the Council](methodology/05_when_to_use.md)
A decision framework with three tiers (Always Council, Council Recommended, Single Model Fine), a cost-benefit analysis with real time estimates, and scaling guidance for 2, 3, and 4-model panels.

## Examples

Real council sessions showing the methodology in action.

### [Architecture Decision](examples/architecture_decision.md)
A full walkthrough of choosing a data pipeline architecture for a trading application. Shows all four model responses, human triage notes, reconciliation, and how an outlier response (Grok challenging latency assumptions) prevented weeks of unnecessary distributed systems work.

### [Code Review Session](examples/code_review_session.md)
Parallel code review of authentication middleware across four models. Each model catches different issues (architecture, edge cases, performance, security). Demonstrates the Complementary Fragments pattern and shows the merged review output.

### [Research Synthesis](examples/research_synthesis.md)
Evaluating frontend frameworks for a client-facing dashboard. Shows how to build a weighted decision matrix from model outputs, handle contradictory recommendations, and arrive at a high-confidence decision with specific implementation guidance.

## Templates

Ready-to-use templates for running council sessions. Copy, fill in, and go.

- **[Council Prompt Template](templates/council_prompt_template.md)** - The prompt you send to each model, with variants for code review, research, and debugging tasks
- **[Reconciliation Prompt Template](templates/reconciliation_prompt_template.md)** - Templates for synthesis, triple-audit verification, and contradiction resolution
- **[Session Log Template](templates/session_log_template.md)** - Structured documentation template with a filled example

## Core Principles

These are the non-negotiables. The full list is in [Quality Rules](methodology/04_quality_rules.md).

1. **150% is the floor, 200% is the target.** Council output must be measurably better than the best single-model response.
2. **Triple-audit before locking.** Independent analysis, reconciliation, then adversarial review.
3. **No single model owns the truth.** The human orchestrator is the final authority.
4. **Consensus is signal, not proof.** Models share training data. They can all be wrong the same way.
5. **Document every decision.** Rationale, alternatives considered, dissenting views preserved.
6. **Export before context gets heavy.** Context windows degrade. Export early, export often.

## Built With This Method

The AI Council Method produced these projects:

- **[Baseline Showcase](https://github.com/Preston2012/baseline-showcase)** - Full-stack application demonstrating production-quality AI-assisted development
- **[Trading Toolkit](https://github.com/Preston2012/trading-toolkit)** - Trading application with real-time data processing and portfolio management
- **[Portfolio](https://baseline.marketing/built)** - Live portfolio showcasing applications built with this methodology

6 application projects (1 published on app stores, 1 in closed testing, 1 functional prototype, 3 in development/design). 100,000+ lines of code across all projects.

## Contributing

This methodology is actively evolving. If you run council sessions and develop patterns or rules worth sharing, contributions are welcome.

The bar is the same one applied to the methodology itself: real experience, concrete examples, actionable content. No theory without practice.

## Contact

Preston - [preston@baseline.marketing](mailto:preston@baseline.marketing)

## License

[MIT](LICENSE)


---

## Methodology in use

This methodology is not theoretical. It is in active use across:

- **[Demiurge](https://github.com/Preston2012/demi).** Open-source memory engine for AI agents. Every architectural decision, prompt iteration, and benchmark configuration runs through council reconciliation.
- **[Ouroboros](https://github.com/Preston2012/ouroboros).** Governance architecture document. Drafted, audited, and refined via council reconciliation across multiple model families.
- **[Baseline](https://github.com/Preston2012/baseline-showcase).** Political intelligence platform. Built solo using this methodology.

Coming next: **[Memory Sovereignty Principles](https://github.com/Preston2012/memory-sovereignty-principles).** A spec + benchmark for personal AI memory, drafted and audited via council.

[Preston Winters](https://github.com/Preston2012). Solo. [preston@baseline.marketing](mailto:preston@baseline.marketing).
