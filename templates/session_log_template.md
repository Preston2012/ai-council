# Session Log Template

Logging every council session is how you build institutional memory. Over time, your logs reveal which models excel at which tasks, which patterns produce the best outcomes, and where past decisions need revisiting. This practice aligns with [Quality Rules](../methodology/04_quality_rules.md) Rule 5: always preserve dissenting views and the reasoning behind your final call.

## Log Template

```
# Council Session Log

## Session Info
- **Session ID:** [YYYY-MM-DD-topic-short-name]
- **Date:** [YYYY-MM-DD]
- **Topic:** [Brief description of the question]
- **Models Used:** [e.g., Claude, GPT-4, Gemini, Grok]
- **Session Type:** [Quick (2 models) / Standard (3) / Deep (4)]

## Question Posed
[Paste the exact prompt sent to all models]

## Model Responses

### Model A: [Name]
**Key Points:**
- [Main recommendation]
- [Supporting reasoning]
- [Notable insights]

**Strength:** [What this response did best]
**Weakness:** [Where it fell short]

### Model B: [Name]
[Same structure]

### Model C: [Name]
[Same structure]

### Model D: [Name] (if applicable)
[Same structure]

## Reconciliation

**Pattern Observed:** [Strong Consensus / Majority + Outlier / Split Decision / Universal Uncertainty / Complementary Fragments / Confident Disagreement]

**Convergence Points:**
- [Where models agreed]

**Divergence Points:**
- [Where models disagreed and how it was resolved]

## Final Decision
**Decision:** [Clear statement of what was decided]

**Rationale:** [Why this decision, in 2-3 sentences]

**Alternatives Considered:**
- [Alternative 1 and why it was not chosen]
- [Alternative 2 and why it was not chosen]

**Dissenting Views Preserved:**
- [Any valid concerns that were noted but overridden, and why]

## Action Items
- [ ] [Next step 1]
- [ ] [Next step 2]
- [ ] [Next step 3]

## Context Export Notes
- **Artifacts exported:** [List files, documents, or code exported from this session]
- **Follow-up sessions needed:** [Yes/No, and what topic]
- **Rules generated:** [Any new rules added to the knowledge base from this session]
```

## Filled Example

```
# Council Session Log

## Session Info
- **Session ID:** 2026-03-15-database-selection-ecommerce
- **Date:** 2026-03-15
- **Topic:** Choose primary database for GreenCart e-commerce platform
- **Models Used:** Claude, GPT-4, Gemini
- **Session Type:** Standard (3)

## Question Posed
TASK: Recommend PostgreSQL or MongoDB as the primary database for a new e-commerce platform.

CONTEXT:
- Project: GreenCart, a B2C e-commerce platform selling sustainable household products
- Current state: Greenfield project, no existing data. MVP targets 5,000 SKUs, 50k monthly active users.
- Tech stack: Next.js frontend, Python/FastAPI backend, deployed on AWS
- Constraints: 2 backend engineers (both experienced with SQL, limited NoSQL experience), 4-month timeline to MVP, scaling target of 500k MAU within 18 months

CONSTRAINTS:
- Must support full-text search on product descriptions
- Must handle flexible product attributes (eco-certifications vary by category)
- ACID transactions required for order processing
- Budget: $500/month for managed database services at launch

## Model Responses

### Model A: Claude
**Key Points:**
- Recommends PostgreSQL with JSONB columns for flexible product attributes
- Use pg_trgm and tsvector for full-text search at MVP scale
- ACID compliance is native; no workarounds needed for order processing
- Suggests migrating to Elasticsearch for search only if query volume exceeds 10k searches/hour

**Strength:** Practical migration path with clear thresholds for when to add complexity
**Weakness:** Did not address read-replica strategy for scaling to 500k MAU

### Model B: GPT-4
**Key Points:**
- Recommends PostgreSQL as primary, with a strong case for the team's existing SQL expertise
- Proposes a schema design using EAV (entity-attribute-value) for flexible product attributes
- Recommends RDS Multi-AZ for availability, read replicas at the 200k MAU mark
- Warns against premature optimization toward NoSQL

**Strength:** Addressed the scaling roadmap through 500k MAU in detail
**Weakness:** EAV pattern is a known anti-pattern for query performance; JSONB is simpler and faster

### Model C: Gemini
**Key Points:**
- Recommends MongoDB, citing natural fit for variable product schemas
- Acknowledges ACID concern but notes MongoDB 4.0+ supports multi-document transactions
- Suggests MongoDB Atlas for managed hosting within budget
- Recommends the team invest 2 weeks in NoSQL training before starting

**Strength:** Honest about the team skill gap and proposed a mitigation plan
**Weakness:** Underestimated the performance overhead of multi-document transactions for order processing at scale

## Reconciliation

**Pattern Observed:** Majority + Outlier (2 PostgreSQL, 1 MongoDB)

**Convergence Points:**
- All three models agreed flexible product attributes are the key schema challenge
- All three agreed full-text search is solvable with either stack at MVP scale
- Two of three recommended PostgreSQL, citing team expertise as the deciding factor

**Divergence Points:**
- Gemini recommended MongoDB; resolved by weighing team expertise and timeline risk. Two weeks of NoSQL training on a 4-month timeline is a 12.5% schedule cost.
- Claude recommended JSONB while GPT-4 recommended EAV for flexible attributes; resolved in favor of JSONB after identifying EAV query performance concerns.

## Final Decision
**Decision:** PostgreSQL on AWS RDS, using JSONB columns for flexible product attributes and tsvector for full-text search.

**Rationale:** The team's SQL expertise eliminates ramp-up risk on a tight timeline. PostgreSQL's JSONB provides the schema flexibility MongoDB would offer, without sacrificing ACID transactions or requiring the team to learn a new paradigm.

**Alternatives Considered:**
- MongoDB: Natural schema flexibility but introduces training cost and transaction complexity the team cannot afford in a 4-month window
- PostgreSQL with EAV pattern: Rejected due to known query performance degradation on attribute-heavy filtering

**Dissenting Views Preserved:**
- Gemini's point about MongoDB's natural fit for variable schemas is valid. If GreenCart expands to a marketplace model with thousands of sellers defining custom attributes, revisit this decision. The JSONB approach has limits around deeply nested queries.

## Action Items
- [ ] Set up RDS PostgreSQL Multi-AZ instance on AWS
- [ ] Design product schema with JSONB column for category-specific attributes
- [ ] Implement full-text search with tsvector; benchmark at 5,000 SKUs
- [ ] Document scaling trigger: add read replica at 200k MAU or when p95 read latency exceeds 100ms

## Context Export Notes
- **Artifacts exported:** Schema design draft (schema_v1.sql), RDS cost estimate spreadsheet
- **Follow-up sessions needed:** Yes; search scaling strategy when approaching 10k searches/hour
- **Rules generated:** Added rule: "When team has strong expertise in one stack, default to that stack unless the alternative offers a 3x or greater advantage on the core problem."
```
