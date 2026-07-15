# Book Grader: Architect Interview Context

**Evidence-backed book content analysis**: Analyze actual text, extract specific events with proof, show parents what they're actually getting.

---

## What You're Demonstrating

This project shows:

1. **Pluggable Abstraction** (LLM layer)
   - Core logic is independent of which LLM you use
   - Swap models by changing `.env`, not rewriting pipeline
   - This is interface design working

2. **Deterministic Testing** (Mock provider)
   - Tests run without GPU
   - Same input = same output every time (critical for a rating system)
   - Pipeline logic is separate from model quality

3. **Privacy by Default** (Local-first)
   - Cloud is opt-in, not the default
   - Users control data, not the company
   - Architecture reflects values

4. **Evidence Traceability** (No black boxes)
   - Every rating claim has proof (chapter, excerpt, count)
   - No "trust us" ratings
   - Accountability built into design

---

## Interview Questions I'd Ask

### Architecture & Design

1. **"Why is the mock provider important? Why not just test with the real LLM?"**
   - Tests: Do you understand test speed/isolation vs. model quality?
   - Answer shows: You separate testing concerns from model concerns

2. **"If quality depends on model choice, how did you avoid re-architecting when switching models?"**
   - Tests: Can you design for change? Abstraction discipline?
   - Look for: Provider interface, zero changes to core logic

3. **"Chunking splits text into ~2000-word pieces. What's the downside?"**
   - Tests: Do you know the tradeoff? Can you articulate it?
   - Answer shows: Chunk boundaries might split scenes, but chapter-boundary mitigates

### Implementation

4. **"LLM hallucinates content. How does the system catch that?"**
   - Tests: Do you validate against source? Have audit trails?
   - Look for: Evidence traceability (chapter + excerpt stored alongside rating)

5. **"A parent says 'your rating is wrong, that scene isn't actually violent.' How do they prove it? How do you respond?"**
   - Tests: How does the system build credibility?
   - Answer shows: Evidence (chapter + excerpt) lets parent verify instantly

### Product & Business

6. **"Why local-first instead of cloud-first?"**
   - Tests: Do you understand privacy as a feature, not a cost?
   - Answer shows: Reading history is personal; local keeps data private

7. **"Your reports are ~5 pages with evidence. Why not just output a number (like 4/5)?"**
   - Tests: Do you understand the problem you're solving?
   - Answer shows: 4/5 is useless without evidence

---

## Code Review Checklist

Look for:

- [ ] **Provider abstraction is clean** (adding a new provider requires minimal changes)
- [ ] **Mock provider is deterministic** (same input → same output)
- [ ] **Evidence is stored with ratings** (every claim has a source)
- [ ] **Chunking respects chapter boundaries** (mitigates the split-scene problem)
- [ ] **Config is external** (swapping models is a `.env` change, not code change)
- [ ] **Tests run without models** (mock provider means CI is fast)

---

## What This Says About You

- **Abstraction discipline**: You don't hardcode assumptions
- **Privacy-conscious**: You default to local, not cloud
- **Transparency-first**: Every claim has proof
- **Testable design**: Core logic is independent of LLM choice
- **User empathy**: You understand why parents need evidence, not just ratings

---

**Status**: Architected and designed by me. Implementation accelerated with Claude.

**Contact for code review**: yow.stephend@gmail.com
