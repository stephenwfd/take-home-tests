# Sr. Full Stack Engineer Take-Home Project
## Complete Package for RET Ventures ROP Hiring

---

## What You've Built

A **production-grade take-home project** that tests full-stack engineering capabilities: React, TypeScript, Node.js, PostgreSQL, API design, and scalability thinking.

**Feature:** Renewal Risk Detection System for ROP  
**Time Limit:** 2 hours  
**Complexity:** Medium (tests all the right things without being impossible)

---

## Package Contents

### 1. **renewal_risk_takehome.md** ← START HERE
The main project brief that candidates receive. 
- Business context
- 5 requirements (schema, API, dashboard, webhooks, error handling)
- Evaluation criteria
- Submission format

**Length:** ~1,200 words (candidates read in 5 min, build for 2 hours)

---

### 2. **starter_schema.sql**
Pre-built PostgreSQL schema with core ROP tables.
- Properties, Units, Residents, Leases, Pricing, Ledgers
- Candidates only design the renewal risk schema
- Saves ~30 min of boilerplate

---

### 3. **evaluation_rubric.md** ← SCORING GUIDE
Detailed rubric for evaluating submissions.
- 100-point scale
- Backend (60 pts): Data modeling, API, webhooks, query performance
- Frontend (25 pts): Functionality, UX
- Code quality (15 pts): Clarity, error handling
- Bonus agentic development (5 pts)

**Includes:**
- Red flags (auto-fail)
- Green flags (strong hire signals)
- Deep-dive questions for Round 3 interview
- FAQ for evaluation consistency

---

### 4. **seed_and_testing.md**
Manual testing guide and seed script.
- TypeScript seed script (copy-paste ready)
- Step-by-step testing workflow
- Expected results
- Troubleshooting
- Time-saving tips

**Use this to:**
- Verify candidates' submissions work
- Know how to test webhooks (webhook.site integration)
- Debug if something breaks

---

### 5. **HIRING_GUIDE.md**
Complete hiring workflow and process.
- How to send to candidates
- How to evaluate (60 min per submission)
- Round 3 interview structure (45 min)
- Decision criteria
- Customization notes
- FAQ

---

### 6. **SAMPLE_EXCELLENT_SUBMISSION.md**
What a ~80-point submission looks like.
- Directory structure
- README (clear, concise)
- Key architectural decisions (schema, webhooks, queries)
- Sample code (controller, service, React component)
- Testing approach

**Use this to:**
- Set expectations ("this is what excellence looks like")
- Train evaluators ("here's how to spot good design")
- Explain scoring to candidates

---

## How to Use This

### Timeline (Example)

**Friday 2pm:** Send `renewal_risk_takehome.md` + `starter_schema.sql` + `seed_and_testing.md` to candidate  
**Saturday 4pm:** Candidate submits GitHub repo  
**Monday 9am:** You evaluate (1.5 hours) using `evaluation_rubric.md`  
**Monday 10:30am:** Schedule Round 3  
**Tuesday 2pm:** 45-min pitch defense interview  
**Tuesday 3pm:** Debrief with team, hire/reject decision  

**Total time:** ~3 hours per candidate (1.5 eval + 0.75 interview + 0.75 debrief)

---

### Step 1: Send to Candidate

Copy the email template from `HIRING_GUIDE.md` and attach:
- `renewal_risk_takehome.md` (the spec)
- `starter_schema.sql` (the foundation)
- `seed_and_testing.md` (how to verify it works)

Set a 2-hour timer.

---

### Step 2: Evaluate Submission (60 min)

1. **Clone their repo and run it** (10 min)
   - `npm install` + `npm run dev` (backend + frontend)
   - Does it start without errors?

2. **Run seed data** (5 min)
   - Does the seed script work?
   - Can you see realistic test data?

3. **Test the API** (10 min)
   - Calculate renewal risk (using curl examples from seed_and_testing.md)
   - Dashboard loads data?

4. **Test webhook delivery** (10 min)
   - Click "Trigger Renewal Event"
   - Use webhook.site to verify delivery
   - Does retry logic work if you simulate failure?

5. **Review code** (20 min)
   - Use `evaluation_rubric.md` as a checklist
   - Check schema design (is it efficient? multi-tenant safe?)
   - Check webhook logic (retry, idempotency, DLQ)
   - Check React components (clear, testable)

6. **Score it** (5 min)
   - Use the 100-point rubric
   - Most candidates: 60-80 points
   - 80+: Strong hire
   - 50-70: Good, proceed to Round 3
   - <50: Likely not a fit

---

### Step 3: Round 3 Interview (45 min)

Use the deep-dive questions from `evaluation_rubric.md`:

1. **Opening** (5 min): "Walk me through what you built."
2. **Schema design** (10 min): Why this structure? How does it scale?
3. **Webhooks** (10 min): How do you ensure delivery? Prevent duplicates?
4. **Query performance** (5 min): How do you score 5000 residents efficiently?
5. **API design** (5 min): What about race conditions?
6. **Frontend** (5 min): How would you handle 50k residents?
7. **Pragmatism** (5 min): What would you do differently?

---

### Step 4: Hiring Decision

**Strong hire (80+ points + good Round 3):**
- Clear thinking about scalability
- Solid code quality
- Pragmatism (shipped 80% in 2 hours)
- Can explain decisions
- Honest about AI-generated code

**Weak hire (50-70 points + mediocre Round 3):**
- Code works but design is so-so
- Can't explain why they chose certain patterns
- Missing error handling or edge cases
- Defensive about feedback

**Auto-reject (<50 points or red flags):**
- Feature doesn't work
- Schema is broken (can't query by property)
- No idempotency (webhooks delivered multiple times)
- Dishonest about work
- Unreadable code

---

## Key Features of This Package

### ✓ Comprehensive
- Covers full-stack (React, Node, PostgreSQL)
- Tests scalability thinking (webhooks, retries, idempotency)
- Tests pragmatism under pressure (2-hour deadline)

### ✓ Fair
- Starter schema saves 30 min of boilerplate
- 2-hour deadline is tight but doable
- Ambiguity is intentional (evaluates decision-making)

### ✓ Consistent
- Rubric ensures all evaluators score the same way
- Red flags are clear (auto-fails)
- Green flags are specific (what excellence looks like)

### ✓ Efficient
- 60 min evaluation per candidate
- 45 min interview (focused questions)
- Clear decision framework

---

## What You're Actually Testing

By the end, you'll know:

1. **Can they ship?** Do they build working features end-to-end?
2. **Can they think about hard problems?** (Webhooks, retries, idempotency—not just UI)
3. **Are they pragmatic?** Do they make reasonable tradeoffs?
4. **Can they communicate?** Do they explain their thinking clearly?
5. **Do they understand databases?** Schema design, query performance, multi-tenancy
6. **Can they evolve?** Do they admit unknowns? Respond to feedback?

---

## Customization

### Want to change the feature?
Not recommended—the renewal risk feature tests all the right things. But if you must:
- Make sure it has similar complexity (async delivery, data modeling, UI)
- Rewrite the rubric and testing guide
- Test it yourself first

### Want to adjust time?
- 2 hours is intentionally tight (forces pragmatism)
- 3 hours: allows more polish, same quality signal
- 90 min: some candidates will run out of time on webhooks (might be okay)

### Want different weights?
- Currently: 60% backend, 25% frontend, 15% code quality
- If you want to emphasize frontend: bump to 40% backend, 40% frontend
- If you want to emphasize systems thinking: add a "scaling" section (10%)

---

## Red Flags (Auto-Fail)

If you see any of these, reject:

1. **Webhook state is in memory** → Will lose state on restart
2. **No idempotency** → Webhooks delivered multiple times
3. **SQL injection** → Concatenating user input into queries
4. **Dishonesty** → Claims they wrote code they didn't, or hides AI usage
5. **Broken schema** → Can't query residents by property
6. **Unreadable code** → No one could maintain it

---

## Green Flags (Strong Hire)

Look for these signals:

1. **Thoughtful schema design** with clear rationale
2. **Bulletproof webhook delivery** (retry + backoff + idempotency + DLQ)
3. **Performance-aware queries** (no N+1, proper indexes)
4. **Clean, organized code** (clear separation of concerns)
5. **Good error handling** and edge case thinking
6. **Honest about AI** (identifies what was generated and why)
7. **Clear documentation** (README, schema decisions, testing instructions)
8. **Minimal but functional UI** (not over-designed)

---

## FAQ

**Q: How long does evaluation take?**  
A: 60 minutes per submission (10 min setup, 30 min testing, 20 min code review)

**Q: Should I hire based on this alone?**  
A: No. This is Round 2. Do Round 1 (phone screen) and Round 3 (pitch defense) too.

**Q: What if they run out of time?**  
A: That's fine. Look at what they shipped. Unfinished but clean code beats half-broken complete code.

**Q: What if they use Claude Code for everything?**  
A: That's okay IF they identify what was generated and show they understand it. In Round 3, ask them to explain the webhook retry logic. If they can't, red flag.

**Q: How do I know if they cheated?**  
A: In Round 3, ask detailed questions about specific code. If they can't explain it, they probably didn't write it.

---

## Next Steps

1. **Review this package** (15 min)
2. **Customize if needed** (schema, time limit, weights)
3. **Test the project yourself** (run through it once, 60 min)
4. **Send to first candidate** (use email template from HIRING_GUIDE.md)
5. **Evaluate using rubric** (60 min per candidate)
6. **Interview strong candidates** (45 min, use deep-dive questions)
7. **Make hiring decision** (30 min debrief with team)

---

## Files in This Package

```
00_READ_ME_FIRST.md                 ← You are here
renewal_risk_takehome.md            ← Send to candidates
starter_schema.sql                  ← Send to candidates
seed_and_testing.md                 ← Send to candidates
evaluation_rubric.md                ← Use for scoring
HIRING_GUIDE.md                     ← Full hiring workflow
SAMPLE_EXCELLENT_SUBMISSION.md      ← Reference for what's good
```

---

## Questions?

Everything is designed to be self-contained. If something is unclear:
1. Check `HIRING_GUIDE.md` (FAQ section)
2. Check `evaluation_rubric.md` (deep-dive section)
3. Review `SAMPLE_EXCELLENT_SUBMISSION.md` (what does good look like?)

Good luck with hiring. This is a solid test.
