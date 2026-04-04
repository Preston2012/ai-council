# Example: Architecture Decision Council Session

This walkthrough covers a real council session for a data pipeline architecture decision. It shows every step: the prompt, the raw model responses, the triage, the reconciliation, and the final call. Follow along to see how the process works in practice.

---

## The Scenario

You're building [trading-toolkit](https://github.com/Preston2012/trading-toolkit), a trading application that needs to process market data, execute trades, and update portfolio positions in near-real-time. Your team is small (2-3 engineers). The system needs to handle bursts of high-frequency data during market hours, then go mostly idle after close.

The question on the table: what architecture should the data pipeline use?

This is a high-stakes, hard-to-reverse decision, so you run a full four-model council session.

---

## Step 1: The Prompt

You send the following prompt, unchanged, to all four models:

```
TASK: Recommend the data pipeline architecture for a trading application
that processes market data, executes trades, and updates portfolio
positions in near-real-time.

CONTEXT:
- Project: trading-toolkit, a self-hosted trading application for
  active retail traders
- Current state: Early development. No existing pipeline. We have a
  working prototype with synchronous REST calls to broker APIs, but it
  cannot keep up during high-volume market periods
- Tech stack: Python 3.12, FastAPI, PostgreSQL 16, Redis (used for
  session caching only), deployed on a single 8-core VM
- Constraints: Team of 2-3 engineers, no dedicated DevOps, budget
  under $500/month for infrastructure

CONSTRAINTS:
- Must handle 5,000+ market data events per second during peak
  (market open, major news events)
- Baseline throughput is ~200 events/second during normal trading hours
- Latency target: process and act on market data within 500ms
- Must support replay of historical data for backtesting
- Cannot require Kubernetes or complex orchestration (no DevOps staff)
- Must run reliably on a single server with the option to scale out later

OUTPUT FORMAT:
- A clear architecture recommendation with justification
- Component diagram or description of data flow
- Technology choices for each layer (ingestion, processing, storage,
  execution)
- Risks, tradeoffs, and migration path if the system outgrows the
  initial design
- Estimated operational complexity (1-10 scale)

NOTE: This is an independent analysis. Do not reference or assume input
from other AI models. Provide your own assessment based solely on the
information given.
```

Notice how specific this is. You include the event rates, latency target, team size, and infrastructure constraints. Vague prompts produce vague answers.

---

## Step 2: Model Responses (Abbreviated)

### Claude

Claude focuses on separation of concerns and reliability patterns.

- Recommends a fully event-driven architecture with four independent services: ingestion, processing, execution, and reporting
- Proposes Redis Streams as the message backbone (you already have Redis deployed; Kafka is overkill for this scale)
- Each service reads from and writes to named streams, enabling independent scaling and failure isolation
- Emphasizes replay capability: Redis Streams natively supports consumer groups and message persistence, which maps directly to the backtesting requirement
- Suggests using Python's `asyncio` for all services, keeping the stack homogeneous
- Warns explicitly against over-engineering: "For a 2-3 person team, operational simplicity is a feature, not a compromise"
- Proposes a thin orchestration layer using `supervisord` instead of Docker Compose or Kubernetes
- Recommends PostgreSQL for trade records and portfolio state, Redis for real-time event flow
- Rates operational complexity at 5/10
- Includes a migration path to Kafka if throughput exceeds 50k events/second

### GPT-4

GPT-4 takes a pragmatic, incremental approach.

- Suggests a hybrid architecture: monolithic core application with event-driven hooks specifically for the market data pipeline
- Argues that decomposing into four services from day one is premature for a team this size
- Proposes keeping trade execution, portfolio management, and reporting inside a single FastAPI application
- Only the data ingestion layer should be separated, feeding into the monolith via a lightweight message queue
- Recommends RabbitMQ for the message queue, citing its maturity, Python client quality, and lower operational overhead compared to Kafka
- Provides a concrete three-phase migration path: (1) monolith with async ingestion, (2) extract execution engine when latency requires it, (3) full service decomposition only if team grows to 5+ engineers
- Emphasizes that the 500ms latency target is easily achievable without distributed systems
- Suggests adding a circuit breaker pattern for broker API calls
- Recommends PostgreSQL with partitioned tables for historical data
- Rates operational complexity at 3/10 for phase 1, scaling to 6/10 at phase 3

### Gemini

Gemini delivers a data-layer-focused analysis with detailed numbers.

- Proposes a dual-mode architecture: streaming pipeline for real-time market data, batch processing for end-of-day reconciliation and reporting
- Recommends Apache Flink for the streaming layer, citing its exactly-once processing guarantees and windowing capabilities
- For batch processing, suggests a simple cron-triggered Python script that queries the database and generates reports
- Provides detailed latency analysis: at 5,000 events/second with 200-byte average payload, throughput is ~1MB/s, well within single-node capacity
- Estimates p99 processing latency at 50-80ms with Flink, leaving 420ms of budget for execution
- Strongly recommends PostgreSQL with the TimescaleDB extension for time-series market data storage
- Explains why TimescaleDB matters: automatic partitioning by time, 10-100x faster time-range queries compared to standard PostgreSQL, native continuous aggregates for OHLCV candle computation
- Provides schema design for the `market_data` hypertable with suggested chunk intervals
- Notes that Flink requires JVM infrastructure, which adds operational weight to a Python-native team
- Rates operational complexity at 6/10

### Grok

Grok challenges the framing of the question itself.

- Opens with: "Before picking an architecture, let's question the requirements. You say 'near-real-time,' but what does that actually mean for your use case?"
- Points out that 500ms latency is not "near-real-time" in the trading world; HFT systems operate at microseconds, institutional systems at single-digit milliseconds
- At 500ms, you are firmly in the "fast enough for a retail trading tool" category, which does not require any distributed systems infrastructure
- Argues that a well-optimized monolith with async workers (Celery with Redis broker, or even just `asyncio.TaskGroup`) handles 5,000 events/second without breaking a sweat on an 8-core machine
- Asks the key question: "Are you building a trading platform or a trading tool? Because the architecture for each is fundamentally different."
- If it's a tool (which the requirements suggest), recommends: single FastAPI app, `asyncio` for concurrent data ingestion, PostgreSQL for everything, Redis for caching hot data
- Suggests using Python's `multiprocessing` for CPU-bound tasks like indicator calculation, keeping the deployment trivially simple
- Notes that Kafka, Flink, RabbitMQ, and even Redis Streams add operational surface area that a 2-person team will spend weekends debugging instead of building features
- Recommends: "Ship the simplest thing that meets the requirements. You can always add complexity. You can never remove it easily."
- Rates operational complexity at 2/10

---

## Step 3: Human Triage

Here is how you read the room after collecting all four responses.

**Pattern identification:** This is a Majority + Outlier (Pattern 2). Claude, GPT-4, and Gemini all assume some form of distributed or multi-component processing is needed. Grok disagrees fundamentally with the premise.

**Triage notes:**

- Three models propose varying degrees of distributed architecture. They disagree on implementation details (Redis Streams vs. RabbitMQ vs. Flink) but agree on the general shape.
- Grok's challenge is uncomfortable but valid. The stated requirements (5k events/sec, 500ms latency, single server) genuinely do not require distributed systems.
- Grok's reframing question ("platform or tool?") cuts to the core. trading-toolkit is a tool. The name even says so.
- However, the three distributed-leaning models raise legitimate concerns that a pure monolith ignores: replay capability, failure isolation, and future scaling.
- GPT-4's hybrid approach sits closest to Grok's position while still addressing the concerns Claude and Gemini raise.
- Gemini's TimescaleDB recommendation is strong regardless of architecture choice. Time-series data storage is an orthogonal concern.
- Gemini's Flink suggestion is the weakest recommendation: it introduces JVM infrastructure into a Python-native team, adding significant operational burden for marginal benefit at this scale.
- Claude's interface design guidance (services communicating through well-defined boundaries) is good software engineering even inside a monolith.

**Verdict:** The outlier is right on framing. The majority is partially right on specifics. Time for reconciliation.

---

## Step 4: Reconciliation

You send the following reconciliation prompt to Claude, summarizing what you found:

```
I ran a council session on the data pipeline architecture for
trading-toolkit. Here's what I found:

MAJORITY POSITION (Claude, GPT-4, Gemini):
Some form of distributed/event-driven processing is needed. They
disagree on specifics (Redis Streams, RabbitMQ, or Apache Flink)
but agree the system should have separated components.

OUTLIER POSITION (Grok):
The requirements (5k events/sec, 500ms latency, single 8-core
server) do not actually require distributed systems. A
well-optimized monolith with async workers handles this easily.
Key reframe: "This is a trading tool, not a trading platform."

KEY DATA POINTS:
- Gemini's latency analysis confirms single-node feasibility
  (~1MB/s throughput, p99 under 80ms)
- GPT-4 proposed a phased migration path from monolith to services
- Gemini strongly recommends PostgreSQL + TimescaleDB for
  time-series data
- Claude emphasized designing internal interfaces as if services
  could be extracted later

MY ASSESSMENT:
The outlier reframe is correct. We're building a tool, not a
platform. But I don't want to ignore the valid concerns about
replay capability and future scaling.

Synthesize these positions into a single actionable architecture
recommendation for trading-toolkit.
```

### Claude's Reconciliation Response

Claude synthesizes the following recommendation:

**Architecture: Modular Monolith with Async Data Pipeline**

1. **Single FastAPI application** as the core, with clearly separated internal modules (not services): ingestion, processing, execution, portfolio, and reporting. Each module communicates through well-defined internal interfaces (abstract base classes, typed data objects). This costs nothing in operational complexity but preserves the option to extract any module into a service later.

2. **PostgreSQL with TimescaleDB extension** for all persistent storage. Use hypertables for market data (automatic time-based partitioning), regular tables for trades and portfolio state. This gives you the time-series query performance Gemini identified without adding a separate data store.

3. **`asyncio` for concurrent data ingestion.** At 5,000 events/second, Python's async I/O handles this without breaking a sweat. No need for Celery, no need for a separate message queue. Use `asyncio.TaskGroup` for structured concurrency.

4. **Redis for caching and simple pub/sub.** You already have it deployed. Use it for caching hot market data (latest prices, positions) and for lightweight internal pub/sub between modules when you need decoupled notifications. Do not invest in Redis Streams or consumer groups yet.

5. **Replay via database, not event log.** Store raw market data in TimescaleDB. Replay for backtesting means querying historical data and feeding it through the same processing pipeline. This is simpler than maintaining a separate event log and achieves the same result.

6. **Scaling trigger:** Revisit this architecture when either (a) throughput consistently exceeds what a single optimized instance can handle, or (b) the team grows beyond 4 engineers. At that point, GPT-4's phased migration path applies: extract the ingestion layer first, then the execution engine.

**Operational complexity: 2-3/10.**

---

## Step 5: The Final Decision

**Chosen architecture:** Modular monolith with async data pipeline, PostgreSQL + TimescaleDB, Redis for caching and pub/sub.

**Why this won:**

- **Grok's reframe shifted the entire conversation.** The question was never "which distributed architecture?" It was "do we need a distributed architecture at all?" The answer, for a 2-3 person team building a trading tool on a single server, is no.
- **Gemini's data layer recommendation was adopted wholesale.** TimescaleDB is the right answer for time-series market data regardless of application architecture. The hypertable schema design and continuous aggregates for OHLCV candles save significant development time.
- **GPT-4's migration path was documented for future reference.** When (not if) the system outgrows a monolith, the three-phase extraction plan provides a clear roadmap. It lives in the project's `docs/architecture/scaling-plan.md`.
- **Claude's interface design principles were incorporated as guard rails.** Internal modules use abstract interfaces. This is not premature abstraction; it is basic clean architecture that happens to make future extraction possible.

**What was explicitly rejected:**

- Apache Flink (Gemini): JVM dependency is a non-starter for a Python-native team of this size
- Full event-driven architecture (Claude's initial recommendation): correct pattern, wrong scale
- RabbitMQ (GPT-4): unnecessary infrastructure for a single-server deployment

---

## Lessons from This Session

**The outlier was the most valuable response.** Three models dove straight into implementation details for distributed systems. Grok stepped back and questioned whether distributed systems were needed at all. Without the council, you likely would have spent 2-3 weeks setting up Redis Streams or RabbitMQ, only to realize later that `asyncio` did the job.

**"What are you actually building?" is the most useful question in architecture.** Platform decisions require platform architecture. Tool decisions require tool architecture. Mixing them up is one of the most common (and expensive) mistakes in software engineering. Grok's "trading platform or trading tool?" question is the kind of reframe you run a council to get.

**The council prevented weeks of unnecessary work.** Not just the distributed systems setup, but the ongoing operational burden. A RabbitMQ or Kafka deployment that a 2-person team has to maintain on weekends is a cost that compounds forever. The council caught this before the first line of infrastructure code was written.

**Reconciliation is where the value crystallizes.** No single model produced the final answer. The final architecture combines Grok's framing, Gemini's data layer, GPT-4's migration strategy, and Claude's interface design principles. That combination only emerges through the reconciliation step.

For more on how to identify and handle the Majority + Outlier pattern, see [Reconciliation Patterns](../methodology/03_reconciliation_patterns.md).
