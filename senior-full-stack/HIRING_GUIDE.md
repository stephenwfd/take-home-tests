# Sr. Full Stack Engineer Take-Home Project
## Complete Package for RET Ventures ROP Hiring

This package contains everything you need to evaluate senior full-stack engineers for the ROP (Residential Operating Platform) team.

---

## Package Contents

### 1. **renewal_risk_takehome.md** (Main Project Brief)
- **What it is:** The project spec that candidates receive
- **Length:** ~1,200 words
- **What it covers:**
  - Context and business problem
  - 5 main requirements (schema design, API, dashboard, webhooks, error handling)
  - Tech stack expectations
  - Evaluation criteria
  - What NOT to do
  - Submission format

**How to use:** Send this to candidates as the main deliverable. Set a timer for 2 hours.

---

### 2. **starter_schema.sql** (Database Foundation)
- **What it is:** Pre-built PostgreSQL schema with core ROP tables
- **What's included:**
  - Properties, Units, Unit Types
  - Residents, Leases
  - Unit Pricing, Renewal Offers, Resident Ledger
  - Commented placeholders where candidates should add schema

**How to use:** Provide this to candidates so they don't waste time rebuilding core tables. They only design the renewal risk schema.

---

### 3. **evaluation_rubric.md** (Scoring & Interview Guide)
- **What it is:** Detailed rubric for evaluating submissions
- **Sections:**
  - 100-point scoring breakdown
  - Backend (60 points): Data modeling, API, webhooks, query performance
  - Frontend (25 points): Functionality, UX
  - Code quality (15 points): Clarity, error handling, testing mindset
  - Bonus agentic development (5 points)

**Key features:**
- Red flags (auto-fail conditions)
- Green flags (strong signals)
- Deep-dive questions for Round 3 (pitch defense)
- What we're actually hiring for

**How to use:** 
- Use the rubric during evaluation (check off points)
- Use the deep-dive questions in Round 3 (45-min interview after submission)
- Share green flags with team so everyone evaluates consistently

---

### 4. **seed_and_testing.md** (Manual Testing Guide)
- **What it is:** Step-by-step instructions for testing the feature end-to-end
- **Includes:**
  - TypeScript seed script (copy-paste ready)
  - Manual testing workflow (6 steps)
  - Expected results
  - Troubleshooting
  - Time-saving tips

**How to use:**
- Send to candidates so they know how to test
- Use yourself to verify submissions before Round 3
- Reference when debugging if a candidate's feature seems broken

---

## How to Use This Package

### Phase 1: Sending to Candidates

**Email template:**
```
Subject: Sr. Full Stack Engineer Take-Home Project – ROP Renewal Risk System

Hi [Candidate],

We're excited to move to the technical phase of our interview process.

Attached is a 2-hour take-home project. You'll be building a renewal risk detection system for our Residential Operating Platform (ROP)—a critical feature that helps property managers identify residents at risk of not renewing before it's too late.

The project tests:
- Full-stack capabilities (React + Node.js + PostgreSQL)
- Thinking about scalability (webhooks, retries, idempotency)
- Pragmatism under time pressure (ship something that works vs. perfect)

**Files:**
- renewal_risk_takehome.md: The main spec (READ THIS FIRST)
- starter_schema.sql: Pre-built database schema (provided to save you time)
- seed_and_testing.md: Testing guide (so you know how to verify it works)

**Deliverable:**
- A GitHub repository (public or private) with:
  - Working backend (Node + TypeScript + PostgreSQL)
  - Working React dashboard
  - Seed data + testing instructions
  - README explaining your decisions

**Submission:**
- Reply with the GitHub link by [DATE/TIME]
- We'll review and schedule a 45-min pitch defense interview

**Notes:**
- You're expected to use AI tools (Claude, Cursor, etc.) if helpful. Just identify which parts.
- Focus on shipping a working feature, not perfection.
- Ambiguity is intentional—make reasonable decisions and document them.

Good luck! Questions? Reply here.

—[Your Name]
```

---

### Phase 2: Evaluating Submissions

**Timeline:** 
- Candidate submits on Friday
- You evaluate by Monday morning (1-2 hours per submission)
- Schedule pitch defense for Tuesday/Wednesday

**Evaluation process:**
1. **Clone their repo** and run `npm install && npm run dev` (backend + frontend)
2. **Check the README** for setup instructions (5 min)
3. **Run the seed script** and verify data loads (5 min)
4. **Test the API** manually using the curl examples in seed_and_testing.md (10 min)
5. **Test the dashboard** in browser (5 min)
6. **Review the code** using the rubric (30 min)
7. **Score it** (use the 100-point scale)
8. **Note questions** for Round 3

**Time estimate:** 60 minutes per submission

**Scoring guidance:**
- Most candidates will score 60-80 points
- Above 80 = strong candidate, hire unless other concerns
- Below 50 = likely not a fit, polite rejection
- 70-80 = good candidate, proceed to Round 3

---

### Phase 3: Round 3 Interview (45 min)

**Agenda (45 min total):**
1. **Opening** (5 min): "Walk me through what you built and why."
2. **Data modeling deep dive** (10 min): See rubric for specific questions
3. **Webhook delivery deep dive** (10 min): See rubric for specific questions
4. **Query performance** (5 min): See rubric for specific questions
5. **API design** (5 min): Edge cases, race conditions
6. **Frontend** (5 min): Scaling, UX decisions, mistakes
7. **Pragmatism & tradeoffs** (5 min): What would they do differently?

**Evaluation during interview:**
- Can they explain their decisions clearly?
- Do they admit unknowns or defend bad decisions?
- How do they respond to challenging questions?
- Can they think out loud about scaling?

**Use the rubric's deep-dive questions.** Don't make up new ones—consistency matters.

---

### Phase 4: Decision

**Strong hire signals:**
- Submission scores 75+
- Explains decisions clearly in Round 3
- Admits unknowns ("I didn't get to X, but here's how I'd approach it")
- Shows pragmatism (ships 80% of feature in 2 hours rather than 20% perfectly)
- Clear code and thoughtful error handling
- Webhooks are well-designed with idempotency

**Pass signals:**
- Submission scores 50-74
- API works but schema could be better designed
- Webhooks present but maybe not bulletproof
- Dashboard functional but UX is unclear
- Code is readable but maybe missing edge cases

**Reject signals:**
- Submission scores < 50
- Feature doesn't work
- Schema is fundamentally broken
- No idempotency (webhooks delivered multiple times)
- Code is unreadable or dishonest about work

---

## Customization Notes

If you want to modify the project:

### Change the feature (not recommended)
- The renewal risk feature tests all the right things: data modeling, async delivery, API design, React
- Changing it would require rewriting the rubric and testing guide
- If you must change it, make sure the new feature has similar complexity

### Adjust the time limit
- 2 hours is intentionally tight. It forces pragmatism.
- If you increase to 3 hours, expect more polish but not fundamentally different results
- If you decrease to 90 min, some candidates will run out of time on webhooks

### Adjust the scoring
- The rubric is weighted towards backend (60 points) because this is a backend-heavy role
- If you want to emphasize frontend more, adjust the weights
- Keep the bonus agentic development signal to get honest feedback on AI usage

### Add more seed data
- 15 residents is enough to test. 50+ is unnecessary and just makes the test take longer.

---

## FAQ

**Q: Should I hire based on this alone?**
A: No. This is Round 2. You should also do:
- Round 1: Phone screen (culture fit, motivation, experience)
- Round 2: This take-home (technical capability)
- Round 3: Pitch defense + deep questions
- Round 4 (optional): System design conversation (optional for borderline candidates)
- References: Check if they shipped things before

**Q: What if they use Claude Code or Cursor for the entire thing?**
A: That's fine, IF they clearly identify what was generated and show they understand it. In Round 3, ask them to explain the webhook retry logic in detail. If they can't, they don't understand their own code (red flag).

**Q: What if they don't finish?**
A: That's okay. Look at what they did finish:
- Did they prioritize the hard parts (webhooks) or the easy parts (UI)?
- Is the code they wrote high-quality?
- Is there a clear plan for the unfinished parts?

A candidate who ships 80% of the feature with great code beats someone who has 100% of the code but it's half-broken.

**Q: How do I know if they cheated?**
A: In Round 3, ask them to explain specific parts of their code in detail. If they can't explain how their webhook retry logic works, they probably didn't write it or don't understand it. Also ask: "Which parts did you generate with AI and why?" Dishonesty = auto-reject.

**Q: Should I let them ask questions during the take-home?**
A: Yes. Questions show they're thinking. If they ask "how would I query 5000 residents efficiently?"—that's a good question. You can say "great question, that's something you should think about in your design." Don't give them the answer.

**Q: What if their schema is totally different from what I expected?**
A: That's fine! In fact, it's good—it shows they thought about it. In Round 3, ask them why they designed it that way. As long as it works and is well-reasoned, it's a good design.

**Q: Should I require tests?**
A: No. Testing is nice but not required for 2 hours. You're testing their ability to ship, not their ability to write test suites. If they add tests, that's a green flag. If they don't, that's fine.

---

## Red Flags to Watch

During evaluation:

1. **Webhook state is in memory** → Will lose state on restart, auto-reject
2. **No idempotency** → Webhooks delivered multiple times, auto-reject
3. **SQL injection** → Concatenating user input into queries, auto-reject
4. **Schema is broken** → Can't query residents by property, auto-reject
5. **Dishonesty** → Claims they wrote code they didn't, auto-reject

---

## Timeline Example

**Friday 2pm:** Send take-home to candidate  
**Saturday 4pm:** Candidate submits  
**Monday 9am:** You evaluate (1.5 hours)  
**Monday 10:30am:** Schedule Round 3 for Tuesday 2pm  
**Tuesday 2pm:** Round 3 interview (45 min)  
**Tuesday 3pm:** Debrief with team, make decision  
**Tuesday 4pm:** Send offer or rejection  

Total time per candidate: ~3 hours (1.5 evaluation + 0.75 interview + 0.75 debrief)

---

## Final Notes

This package is designed to:
1. **Test the right things**: Full-stack shipping, scalability thinking, pragmatism
2. **Be fair**: The project is doable in 2 hours; you're not setting them up to fail
3. **Be consistent**: The rubric ensures evaluators score the same way
4. **Reveal culture fit**: How they handle ambiguity, tight deadlines, incomplete specs shows a lot

Good luck with hiring. Let me know if you have questions about the rubric or want to customize anything.
