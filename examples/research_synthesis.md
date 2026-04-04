# Example: Research Synthesis Session

This is a real-world example of a council session used to evaluate frontend frameworks for a client-facing analytics dashboard. Four models researched the same question from different angles. The result was stronger than any single model's output.

Follow along to see how the method works in practice.


## The Scenario

You're building a new analytics dashboard for the Baseline platform ([github.com/Preston2012/baseline-showcase](https://github.com/Preston2012/baseline-showcase)). The requirements are clear:

- Real-time data visualization (streaming updates every 2-5 seconds)
- Complex filtering across multiple dimensions
- Responsive design (desktop and tablet)
- The team knows React, but the tech lead wants to evaluate alternatives before committing
- Hard deadline: 8 weeks to a production MVP

The question: should you use React, Vue 3, Svelte, or Angular?


## The Prompt

Sent identically to all four models, using the [council prompt template](../templates/council_prompt_template.md):

```
TASK: Evaluate React, Vue 3, Svelte/SvelteKit, and Angular as the frontend
framework for a new client-facing analytics dashboard. Recommend one with
justification.

CONTEXT:
- Project: Baseline analytics dashboard, a client-facing tool for visualizing
  portfolio performance, risk metrics, and market data
- Current state: Greenfield project, no existing frontend code
- Tech stack: Node.js backend with REST + WebSocket APIs, PostgreSQL, deployed
  on AWS
- Constraints: Team of 4 frontend engineers, all proficient in React, 8-week
  deadline to production MVP

CONSTRAINTS:
- Must render real-time data updates (2-5 second intervals) without jank
- Must support complex filtering (10+ filter dimensions, combinatorial)
- Must be responsive (desktop and tablet, not mobile)
- Must have a mature charting/data visualization ecosystem
- Team is open to learning a new framework if the benefits justify it

RESEARCH SCOPE:
- Compare across: performance, ecosystem maturity, developer experience,
  learning curve, hiring pool, and data visualization library support

DEPTH REQUIRED:
- Enough to make a final framework decision with confidence

OUTPUT FORMAT:
- Analysis of each framework against the requirements above
- Comparison table of key tradeoffs
- A single recommendation with clear justification
- Key risks of your recommendation and how to mitigate them

NOTE: This is an independent analysis. Do not reference or assume input from
other AI models. Provide your own assessment based solely on the information
given.
```


## Model Responses

Each model was given the same prompt. Here is a condensed version of what came back.


### Claude's Analysis: Architecture and Maintainability

Claude approached this as a long-term architecture decision, evaluating how each framework would hold up as the codebase grew.

**React:**
Strong recommendation territory. The ecosystem is unmatched for dashboard tooling: dozens of production-grade component libraries, mature patterns for complex state management, and your team already knows it. The main drawback is state management fragmentation. You will need to choose between Redux Toolkit, Zustand, Jotai, and React Context, and each choice shapes the codebase differently. For a dashboard with heavy cross-component state (filters affecting multiple charts simultaneously), this decision matters.

**Vue 3:**
The Composition API is genuinely well-designed for this kind of application. Pinia as the official state management solution eliminates the "which state library" debate entirely. Computed properties and watchers map naturally to dashboard filter logic. The ecosystem is smaller than React's, but it has what you need: good chart library wrappers, solid component libraries like Vuetify and PrimeVue. The risk is that your team loses 1-2 weeks getting comfortable.

**Svelte/SvelteKit:**
The compiled approach produces smaller bundles and the reactivity model is elegant. Writing `$: filtered = data.filter(...)` instead of `useMemo` with dependency arrays is a real productivity gain. But the ecosystem is still maturing for enterprise dashboard use cases. Fewer battle-tested chart library integrations, fewer UI component kits, and debugging tooling is not yet on par with React DevTools.

**Angular:**
Full batteries-included framework with strong opinions about project structure. TypeScript-first, built-in dependency injection, opinionated routing. Great for large teams (20+ engineers) that need strict conventions. For a team of 4 on an 8-week timeline, the boilerplate and ceremony overhead does not pay off. Angular's strength is governance at scale, and this project is not at that scale.

**Claude's recommendation:** React. The 8-week deadline is the deciding factor. Learning a new framework, even one as approachable as Vue, costs real time when the team also needs to learn the domain (financial data visualization has edge cases). Use the time savings to build better abstractions.


### GPT-4's Analysis: Developer Experience and Shipping Speed

GPT-4 focused on velocity: how fast can the team go from zero to production?

**React:**
Fastest path to production given the team's existing skills. Zero ramp-up time. The critical decision is the charting stack, not the framework. Evaluated three options in depth:
- *Recharts*: easiest to use, declarative API, covers 80% of standard chart types, but struggles with 10K+ data point renders
- *Victory*: more customizable than Recharts, better animation support, but steeper learning curve
- *D3 wrappers (visx, nivo)*: most flexible, best for custom visualizations, but you're essentially writing D3 with React ergonomics

Recommended stack: Next.js (App Router) + Tremor for dashboard layout components + Recharts for standard charts. This combination gets you a polished dashboard UI in days, not weeks.

**Vue 3:**
Second-fastest option. The docs are excellent and the mental model is straightforward. A React developer can be productive in Vue within 3-5 days. But you lose those 3-5 days times 4 engineers, which is 12-20 person-days of reduced velocity during a sprint where every day counts.

**Svelte:**
If the team already knew Svelte, this would be the recommendation. Less boilerplate means faster iteration. But the learning curve, combined with a thinner ecosystem for data visualization, makes this a risk on an 8-week timeline. Layerchart and Pancake are promising but not as mature as Recharts or Victory.

**Angular:**
Slowest option for a small team. The initial project setup, module architecture, and NgModule ceremony take longer than the equivalent in any other framework on this list. Angular's strengths (enterprise governance, strict typing, built-in testing patterns) pay dividends at month 6, not week 2.

**GPT-4's recommendation:** React with Next.js, Tremor for layout, and Recharts for charts. Start with this stack and only reach for D3 when Recharts cannot handle a specific visualization. This gives you the fastest time-to-demo.


### Gemini's Analysis: Performance and Data Handling

Gemini went deep on benchmarks and runtime performance, particularly for real-time data scenarios.

**React:**
Virtual DOM reconciliation adds overhead when you're updating chart data every 2-5 seconds across multiple components. React 18's concurrent rendering helps (it can interrupt re-renders to prioritize user interactions), but for a dashboard rendering 10K+ data points across 6-8 charts simultaneously, you will feel the cost. Mitigation exists: `React.memo`, `useMemo`, virtualized lists, and Web Workers for data processing. But these are workarounds for a fundamental architectural choice.

**Vue 3:**
Proxy-based reactivity is measurably more efficient than React's reconciliation model for frequent, small updates. When a single data point changes in a dataset, Vue's fine-grained reactivity system knows exactly which DOM nodes to update. React re-renders the entire component subtree by default and relies on memoization to skip unnecessary work. For real-time dashboards with streaming data, this difference is meaningful.

**Svelte:**
Compiled output is 30-40% smaller than equivalent React bundles. Direct DOM manipulation (no virtual DOM) means Svelte is the fastest option for raw rendering performance. In benchmarks with synthetic dashboard workloads, Svelte consistently outperforms React and Vue on initial render and update cycles.

**Angular:**
Change detection can become a bottleneck for real-time data unless you adopt OnPush strategy across all components and manually manage change detection cycles. Zone.js overhead adds latency to every async operation. Signals (introduced in Angular 16+) improve this, but they're still newer and less battle-tested.

**Performance comparison (equivalent dashboard, 8 charts, 5K data points each):**

| Metric | React 18 | Vue 3 | Svelte 4 | Angular 17 |
|---|---|---|---|---|
| Initial load (gzip) | 142 KB | 98 KB | 62 KB | 187 KB |
| Re-render (2s interval) | 12ms | 7ms | 4ms | 15ms |
| Memory at steady state | 48 MB | 38 MB | 31 MB | 54 MB |
| Time to interactive | 1.8s | 1.4s | 1.1s | 2.2s |

**Gemini's recommendation:** Vue 3. Best performance-to-productivity ratio for real-time dashboards. The reactivity model is purpose-built for this kind of application. If performance were the only criterion, Svelte wins. But Vue 3 gives you 80% of Svelte's performance advantage with a much more mature ecosystem.


### Grok's Analysis: Risk and Contrarian View

Grok challenged the framing of the question itself.

**The framework is not the critical decision.**

For a data visualization dashboard, the charting library determines 70% of your user experience. The framework is plumbing. You could build a great dashboard in any of these four frameworks, and a terrible one in any of them, depending on your chart library choice.

The real evaluation should be:

- *D3.js*: Maximum control, steep learning curve, works with any framework. If you need custom financial visualizations (candlestick charts, heatmaps with custom color scales, interactive scatter plots with brushing), D3 is the only option that won't fight you.
- *Apache ECharts*: Full-featured out of the box. Handles 100K+ data points without breaking a sweat. Has wrappers for React, Vue, Angular, and Svelte. If your charts are mostly standard types (line, bar, area, pie, scatter), ECharts is the fastest path to a polished result.
- *Plotly*: Strong for scientific and analytical visualizations. Excellent hover tooltips and interactivity out of the box. Heavier bundle size.
- *Observable Plot*: Newer, lightweight, great API design. But less mature for production dashboard use.

Pick the chart library first, then pick whichever framework has the best wrapper for it.

**The hiring argument nobody is making.**

If Baseline grows and the team expands, React developers are roughly 5x easier to find than Vue developers and 10x easier than Svelte developers. This doesn't matter at team size 4. It matters a lot at team size 15.

**The honest observation.**

If you're running a formal evaluation of four frameworks when your team already knows React, you've already decided to use React. You're looking for permission, or you're worried you're missing something. You're not missing anything. React is fine. Stop debating frameworks and start evaluating chart libraries.

**Grok's recommendation:** React. But the real recommendation is to spend the time you would have spent debating frameworks on building a proof-of-concept with 2-3 chart libraries instead.


## Human Triage Notes

After reading all four responses, here is what stood out.

**Pattern observed:** Majority with Outlier, plus Complementary Fragments. (See [Reconciliation Patterns](../methodology/03_reconciliation_patterns.md) for pattern details.)

Three models recommend React. One (Gemini) makes a strong technical case for Vue 3. This is not a simple 3v1 dismissal. Each model adds something the others missed:

- **Claude** identified the state management fragmentation problem and emphasized maintainability guardrails
- **GPT-4** provided a concrete, ready-to-use stack recommendation (Next.js + Tremor + Recharts)
- **Gemini** delivered performance data that informs specific optimization strategies
- **Grok** reframed the entire question: the chart library is the critical decision, not the framework

The practical concerns (timeline, existing knowledge) outweigh the technical performance advantages. Gemini's data is valid, but the performance gap between React and Vue is something you can mitigate with targeted optimizations. You cannot mitigate a missed deadline.

Grok's hiring point deserves documentation even though it is not relevant today. If Baseline grows, this will matter.


## Weighted Decision Matrix

| Criterion | Weight | React | Vue 3 | Svelte | Angular |
|---|---|---|---|---|---|
| Team familiarity | 25% | 10 | 4 | 3 | 5 |
| Ecosystem and libraries | 20% | 9 | 7 | 5 | 8 |
| Performance (real-time) | 15% | 6 | 9 | 10 | 5 |
| Time to ship | 20% | 10 | 6 | 5 | 4 |
| Hiring pool | 5% | 10 | 5 | 3 | 7 |
| Data viz ecosystem | 10% | 9 | 7 | 4 | 7 |
| Long-term maintainability | 5% | 7 | 8 | 6 | 9 |
| **Weighted Score** | **100%** | **8.75** | **6.30** | **5.10** | **5.80** |


## Reconciled Recommendation

**Framework:** React with Next.js (App Router).

The team knows it, the ecosystem supports it, and the 8-week deadline is non-negotiable. This is the low-risk, high-velocity choice.

**Charting stack** (adopting Grok's chart-library-first thinking):
- Tremor for dashboard layout components (cards, KPI displays, filter panels)
- Recharts for standard chart types (line, bar, area, composed charts)
- D3.js for any custom financial visualizations that Recharts cannot handle

**Performance mitigations** (addressing Gemini's concerns):
- Implement `React.memo` and `useMemo` on all chart components from day one
- Use virtualization (TanStack Virtual) for large data tables
- Offload heavy data transformations to Web Workers
- Profile re-render performance weekly during development, not just before launch

**State management:** Zustand for global filter state. Lightweight, minimal boilerplate, pairs well with React's concurrent features.

**Documented for the future:** If Baseline ever rebuilds the dashboard from scratch, Vue 3 is the recommended alternative. Gemini's analysis of Vue's fine-grained reactivity model for real-time data is sound. Preserve that analysis in the project's architecture decision records.

**Confidence:** HIGH. Strong consensus on the framework choice, with complementary insights that shaped the implementation strategy.


## What the Council Added

No single model would have produced this complete picture.

- **Grok's reframing** ("chart library matters more than framework") changed the implementation approach. Without it, the team might have spent two weeks evaluating frameworks and two days picking a chart library. Now those priorities are inverted.
- **Gemini's performance data** informed specific optimization strategies that would not have been planned otherwise. Knowing that React's reconciliation model is measurably slower for streaming updates means you plan for memoization from the start, not as a fix after users complain.
- **GPT-4's concrete stack recommendation** (Next.js + Tremor + Recharts) saved a round of evaluation. Instead of researching dashboard component libraries, the team got a tested combination.
- **Claude's maintainability analysis** set guard rails for code organization. The state management recommendation (Zustand over Redux) came from weighing Claude's fragmentation concern against the project's scope.

This is the value of the council method. Four perspectives, each authentic to how that model approaches research, combined into a decision that is better-reasoned and more actionable than any single response.
