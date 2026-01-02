# Evaluation Rubric & Interview Guide
## Sr. Full Stack Engineer Take-Home: Renewal Risk Detection

---

## Scoring Framework

Total: 100 points

### Backend Architecture & Implementation (60 points)

#### Data Modeling (15 points)
**What we're evaluating:** Can they design a schema that supports their feature without over-engineering? Do they understand multi-tenancy, query patterns, ACID guarantees?

**Excellent (13-15 points):**
- Schema is normalized, efficient, and extensible
- Clear understanding of multi-tenancy (property_id sharding, row-level isolation)
- Indexes chosen deliberately (on frequently queried columns, not everywhere)
- Acknowledges tradeoffs (denormalization for performance vs. normalization for consistency)
- Schema design doc explains rationale

**Good (10-12 points):**
- Schema is functional and mostly efficient
- Covers multi-tenancy but may not deeply think about isolation
- Reasonable indexes, minor inefficiencies
- Brief explanation of design choices

**Acceptable (7-9 points):**
- Schema works but may be over-normalized or missing indexes
- Multi-tenancy present but not fully thought through
- Minimal documentation

**Poor (0-6 points):**
- Schema is broken, missing key relationships, or unsustainable
- N+1 query patterns evident
- No consideration of multi-tenancy or query performance

**Red Flags:**
- Risk scores stored as VARCHAR or JSON in a way that makes querying impossible
- Webhook state stored in files or memory instead of database
- No indexes on frequently queried columns (property_id, resident_id, status)
- Webhook retry logic relies on application memory (will lose state on restart)

---

#### API Design (15 points)
**What we're evaluating:** Do they build REST APIs that are clear, consistent, and handle errors well?

**Excellent (13-15 points):**
- Clear, RESTful endpoints with sensible paths
- Proper HTTP status codes (200, 201, 400, 404, 500, 503, etc.)
- Validation on all inputs
- Consistent response format
- Error messages are informative
- Endpoint contracts match the requirements

**Good (10-12 points):**
- Mostly RESTful, minor inconsistencies
- Good HTTP status codes, may miss a few edge cases
- Validation present, maybe incomplete
- Response format is consistent

**Acceptable (7-9 points):**
- API works but may not be fully RESTful
- Status codes are basic (mostly 200/500)
- Minimal validation

**Poor (0-6 points):**
- API doesn't work or is poorly designed
- Wrong HTTP verbs or status codes
- No error handling

**What to look for in Round 3 (Pitch Defense):**
- Can they explain their endpoint design choices?
- How do they handle race conditions (e.g., two renewals triggered simultaneously)?
- What happens if the batch job is triggered while it's still running?
- How do they validate that a resident is actually at risk (no tampering)?

---

#### Webhook Delivery System (20 points)
**What we're evaluating:** This is the hardest part. Do they understand async delivery, retries, idempotency, and reliability?

**Excellent (18-20 points):**
- Retry logic with exponential backoff (1s, 2s, 4s, 8s, 16s) implemented correctly
- Dead-letter queue for failed deliveries
- Idempotency key (event_id) prevents duplicate deliveries
- Webhook state is persisted atomically in database
- Request signing (HMAC-SHA256 or similar) documented
- Clear separation of concerns (delivery logic separate from API logic)
- Handles partial failures gracefully (one resident's webhook fails doesn't block others)
- Documentation on how RMS should validate webhooks

**Good (14-17 points):**
- Retry logic works, maybe not perfectly exponential
- DLQ present, may not be fully integrated
- Idempotency present but maybe not bulletproof
- Webhook state in database, may have atomicity issues
- Request signing mentioned but not fully implemented
- Most concerns separated

**Acceptable (10-13 points):**
- Retry logic exists, simple implementation
- DLQ missing or basic
- Idempotency partial or missing
- Webhook state may be in memory or have atomicity issues
- No request signing

**Poor (0-9 points):**
- Retry logic missing or broken
- No DLQ
- No idempotency
- Webhook state not persisted
- Single point of failure

**Red Flags:**
- Webhook state stored in memory (will be lost on restart)
- No exponential backoff (constant retry interval)
- No idempotency (duplicates possible)
- No dead-letter queue (failed deliveries lost forever)
- Retries happen synchronously (blocks the API)
- Request signing is hardcoded or insecure

**What to look for in Round 3:**
- How would they scale webhook delivery if load increases 10x?
- What happens if the RMS endpoint is slow (>2s latency)?
- How do they prevent webhook storms (e.g., if RMS keeps returning 5xx)?
- What's their strategy for detecting stuck webhooks?

---

#### Query Performance (10 points)
**What we're evaluating:** Can they write efficient SQL/ORM code?

**Excellent (9-10 points):**
- Queries are optimized: proper joins, eager loading, no N+1
- Indexes are used correctly
- Batch operations where appropriate (e.g., scoring 5000 residents in one query, not a loop)
- Query execution plan shows index usage

**Good (7-8 points):**
- Queries mostly efficient, minor N+1 issues
- Indexes present, may miss some

**Acceptable (5-6 points):**
- Queries work but may be inefficient
- Possible N+1 or full table scans
- Some indexes missing

**Poor (0-4 points):**
- Queries are slow (full table scans, missing indexes, N+1)
- Would fail with real data (250+ residents)

**What to look for in Round 3:**
- Can they explain their query execution plans?
- How would they optimize the renewal risk calculation for 50,000 residents?
- Do they know how to use EXPLAIN ANALYZE?

---

### Frontend Implementation (25 points)

#### Functionality (15 points)
**What we're evaluating:** Does the dashboard work? Does it display data correctly? Does it handle errors?

**Excellent (13-15 points):**
- Dashboard loads and displays residents correctly
- Filters/sorting works if implemented
- Loading state shown while fetching
- Error states handled (no data, API error, etc.)
- "Trigger Renewal Event" button works and provides feedback
- Refresh/polling works if implemented

**Good (10-12 points):**
- Dashboard works, most features present
- Loading state present
- Basic error handling

**Acceptable (7-9 points):**
- Dashboard works but may be missing error/loading states
- "Trigger Renewal Event" may not give feedback

**Poor (0-6 points):**
- Dashboard doesn't work or is broken
- No error handling
- No loading states

**What to look for in Round 3:**
- How would they handle 5000+ residents in the table (pagination, virtualization)?
- How do they prevent accidental double-clicks on "Trigger Renewal Event"?
- What would they do differently at scale?

---

#### UX (10 points)
**What we're evaluating:** Is it usable? Can a property manager understand what they're looking at?

**Excellent (9-10 points):**
- Clear, intuitive interface
- Property managers would understand immediately what they're looking at
- Visual hierarchy is good (risk tier is obvious)
- Color coding makes sense (red = high, yellow = medium)
- Actions are discoverable

**Good (7-8 points):**
- Mostly clear, minor usability issues
- Property managers would mostly understand it

**Acceptable (5-6 points):**
- Functional but could be clearer
- Some confusion about what data means

**Poor (0-4 points):**
- Confusing, hard to use
- Data presentation is unclear

**What to look for in Round 3:**
- Why did they make the design choices they did?
- What would they test with property managers?

---

### Code Quality (15 points)

#### Clarity & Maintainability (8 points)
**What we're evaluating:** Is code readable and organized?

**Excellent (7-8 points):**
- Code is well-organized (clear folder structure)
- Naming is clear and consistent
- Comments explain "why," not "what"
- Functions are small and focused
- No dead code or half-finished ideas

**Good (5-6 points):**
- Code is mostly clear, minor organization issues
- Mostly good naming, maybe a few unclear variables
- Some unnecessary comments or code

**Acceptable (3-4 points):**
- Code works but is messy
- Naming is confusing, comments are sparse
- Some dead code or unclear structure

**Poor (0-2 points):**
- Code is hard to follow
- Poor organization, bad naming
- Lots of dead code

---

#### Error Handling (4 points)
**What we're evaluating:** Do they think defensively?

**Excellent (4 points):**
- Handles edge cases (lease expired, no market rent, etc.)
- Validation on all inputs
- Graceful degradation
- Considers concurrency issues

**Good (3 points):**
- Most edge cases handled
- Good validation, may miss some

**Acceptable (2 points):**
- Basic error handling, missing some cases
- Minimal validation

**Poor (0-1 points):**
- Minimal error handling
- Will break on edge cases

---

#### Testing Mindset (3 points)
**What we're evaluating:** Can they think about testing even if they don't write tests?

**Excellent (3 points):**
- Code is testable (dependency injection, small functions)
- Seed script allows manual testing
- Mock RMS endpoint for webhook testing
- Clear instructions for testing locally

**Good (2 points):**
- Code is mostly testable
- Seed script present
- Basic testing instructions

**Acceptable (1 point):**
- Code could be tested with effort
- Minimal testing setup

**Poor (0 points):**
- Hard to test
- No seed script or testing instructions

---

### Agentic Development Signal (Bonus, +5 points)

**Excellent (5 points):**
- Clearly identifies which parts were generated (backend/frontend/database)
- Explains tradeoffs: what AI did well, what they refined
- Shows good judgment about when to use AI (boilerplate) vs. write manually (complex logic)
- Provides evidence (chat transcripts, code comments)

**Good (3-4 points):**
- Identifies some AI-generated parts
- Brief explanation of tradeoffs
- Shows reasonable judgment

**Acceptable (1-2 points):**
- Mentions using AI but doesn't clearly identify what
- Minimal explanation

**Poor (0 points):**
- Doesn't mention AI or is dishonest about it

**In Round 3, ask:**
- Which parts did you generate with AI?
- Why did you choose to generate those parts?
- What did you have to fix or refine?
- What would you never let AI generate?

---

## Interview Round 3: Pitch Defense (45 min)

After they submit, conduct a structured interview to dig into their thinking.

### Opening (5 min)
- "Walk me through what you built and why."
- Let them talk uninterrupted. This reveals priorities and clarity of thinking.

### Data Modeling Deep Dive (10 min)
- "Walk me through your schema design. Why did you structure it this way?"
- "How would your schema handle this scenario: a resident has their lease renewed twice (they sign a new lease but don't move out). How does that show up in your schema?"
- "What indexes did you add and why?"
- "What would happen if two batch jobs tried to calculate risk at the same time?"

### Webhook Delivery Deep Dive (10 min)
- "Walk me through your webhook retry logic. What happens if the RMS endpoint is slow?"
- "How do you prevent duplicate deliveries? What if the network request succeeds but the confirmation is lost?"
- "Your p95 requirement is 2 seconds. How do you ensure you don't block the 5% of slow requests?"
- "You get a support ticket: 'resident was sent renewal event twice.' How do you debug this?"

### Query Performance (5 min)
- "Scoring 5000 residents—how does your query work? Walk me through the logic."
- "What happens if it takes longer than 2 seconds?"
- "How would you optimize if it took 30 seconds?"

### API Design (5 min)
- "What happens if someone calls 'trigger renewal event' while the batch job is running?"
- "Your API returns a risk score. How confident should property managers be in that score? What caveats would you document?"

### Frontend (5 min)
- "How would you handle 50,000 residents in the dashboard?"
- "A property manager clicks 'Trigger Renewal Event' twice by accident. What happens?"

### Pragmatism & Tradeoffs (5 min)
- "What would you do differently if you had another 2 hours?"
- "What shortcuts did you take? Are you comfortable with them?"
- "If this feature needed to work at 10x scale (2.5M residents), what would break first?"

---

## What We're Actually Hiring For

By the end of this process, you should understand:

1. **Can they ship?** Do they build working features end-to-end?
2. **Can they grok complexity?** Do they understand multi-tenancy, async operations, eventual consistency?
3. **Do they think about the hard problems?** Webhooks, retries, idempotency—not just the obvious UI.
4. **Can they communicate?** Do they explain their thinking clearly?
5. **Are they pragmatic?** Do they make reasonable tradeoffs under time pressure?
6. **Can they evolve?** Given feedback, do they adapt?

---

## Red Flags (Auto-Fail)

- **Schema is fundamentally broken** (can't query residents by property, no foreign keys, etc.)
- **Webhook delivery is in memory** (will lose state on restart)
- **No idempotency** (webhooks delivered multiple times)
- **SQL injection or security issues** (concatenating user input into queries)
- **Code is completely unreadable** (no one else could maintain it)
- **Dishonesty about AI-generated code** or work not done

---

## Green Flags

- Thoughtful schema design with clear rationale
- Webhook retry logic is bulletproof
- Idempotency is clearly handled
- Code is organized and easy to follow
- Good error handling and edge case thinking
- Honest about what AI generated and why
- Clear, working seed data and testing instructions
- Performance-aware query design
- React dashboard is minimal but functional

---

## Follow-Up Questions for Round 4 (If You Get This Far)

If they're a strong candidate and you want to dig deeper:

1. **Architecture at scale:** "How would this system work for a customer with 500 properties and 500k residents?"
2. **Data consistency:** "The renewal risk score can be stale. How do you decide how often to recalculate?"
3. **Multi-tenancy isolation:** "How confident are you that tenant A can't see tenant B's data?"
4. **Monitoring:** "How would you know if webhooks are stuck or failing silently?"
5. **Agentic development:** "How do you use AI tools in your daily work? What do you trust them with?"
