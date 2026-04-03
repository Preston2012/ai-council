# Council Prompt Template

This is the template you send to each AI model during a council session. Copy the core template below, fill in the bracketed sections, and send the identical prompt to every model. Consistency across models is what makes the independent responses comparable.

## Core Template

```
TASK: [Describe the specific question or problem. Be precise.]

CONTEXT:
- Project: [Project name and brief description]
- Current state: [What exists now, what's working, what's not]
- Tech stack: [Languages, frameworks, infrastructure]
- Constraints: [Budget, timeline, team size, technical limitations]

CONSTRAINTS:
- [List any hard constraints: must use X, cannot use Y, must support Z]
- [Performance requirements, security requirements, compliance needs]

OUTPUT FORMAT:
- [What you want back: recommendation with tradeoffs, code implementation, analysis, etc.]
- [Specify structure: bullet points, numbered list, code blocks, etc.]

NOTE: This is an independent analysis. Do not reference or assume input from other AI models. Provide your own assessment based solely on the information given.
```

## Filled Example

```
TASK: Recommend whether to use WebSockets or Server-Sent Events (SSE) for real-time price updates on a trading dashboard.

CONTEXT:
- Project: RetailTrader, a web-based equities trading dashboard for retail investors
- Current state: REST polling every 2 seconds; users complain about stale prices and high data usage on mobile
- Tech stack: React frontend, Node.js/Express backend, PostgreSQL, deployed on AWS ECS behind an ALB
- Constraints: Team of 3 engineers, must ship in 6 weeks, budget for infrastructure is $2k/month

CONSTRAINTS:
- Must support 10,000 concurrent users at launch
- Must work on mobile browsers (iOS Safari, Chrome Android)
- Cannot require sticky sessions (ALB does not support them in current config)
- 99.9% uptime requirement during market hours (9:30 AM - 4:00 PM ET)

OUTPUT FORMAT:
- A clear recommendation (WebSocket or SSE) with justification
- A comparison table of tradeoffs across: latency, browser support, infrastructure complexity, mobile battery impact, and scaling cost
- Migration path from current REST polling approach
- Key risks and how to mitigate them

NOTE: This is an independent analysis. Do not reference or assume input from other AI models. Provide your own assessment based solely on the information given.
```

## Variation Templates

### Code Review Variant

Add these sections to the core template when requesting code review:

```
CODE TO REVIEW:
[Paste the code snippet or link to the file]

AREAS OF CONCERN:
- [e.g., "Memory usage under load," "Thread safety," "SQL injection risk"]

REVIEW FOCUS:
- [e.g., "Security vulnerabilities," "Performance bottlenecks," "Readability and maintainability"]
- [Specify severity rating scale if desired: Critical / Major / Minor / Nit]
```

### Research Variant

Add these sections when requesting research or analysis:

```
RESEARCH SCOPE:
- [e.g., "Compare managed Kubernetes offerings from AWS, GCP, and Azure"]

DEPTH REQUIRED:
- [e.g., "Executive summary with enough detail to make a purchasing decision"]

SOURCES TO PRIORITIZE:
- [e.g., "Official documentation, published benchmarks, production case studies"]
- [e.g., "Avoid blog posts older than 12 months"]
```

### Debugging Variant

Add these sections when requesting help with a specific bug:

```
ERROR DESCRIPTION:
- [Exact error message, including stack trace if available]
- [When it started occurring, frequency]

STEPS TO REPRODUCE:
1. [Step 1]
2. [Step 2]
3. [Step 3]

WHAT I HAVE TRIED:
- [Attempted fix 1 and result]
- [Attempted fix 2 and result]

ENVIRONMENT:
- [OS, runtime version, relevant dependency versions]
```

## Tips for Effective Prompts

- **Be specific.** "Should we use a cache?" is weak. "Should we add a Redis cache between our API layer and PostgreSQL for the product catalog endpoint that currently averages 800ms p95?" is strong.
- **Include numbers.** Concrete metrics (request volume, latency targets, data sizes) produce concrete recommendations.
- **State constraints explicitly.** Models cannot read your mind about budget limits or team expertise. If your team has never operated Kafka, say so.
- **Keep it identical.** The entire point of the council method is independent analysis. Do not customize the prompt per model.
