# Book Grader: Architectural Decision Records

Evidence-backed content analysis for books: analyze actual text, extract specific events with proof, aggregate deterministically.

---

## ADR-1: Chunked Text Processing (Not Full-Book Prompts)

**Problem**: A 500K-word novel won't fit in one LLM prompt (context limits, latency, cost).

**Decision**: Chunk text into ~2000-word segments, analyze each independently, aggregate results.

**Why**:
- Finite context windows (even Claude has limits)
- Streaming/real-time progress possible
- Parallelizable (chunk N can run while chunk N+1 is encoding)
- Graceful degradation (one chunk fail ≠ whole book fail)

**Trade-off**: Chunk boundaries might split scenes. Mitigation: respect chapter boundaries when possible.

---

## ADR-2: Deterministic Mock Provider for Testing

**Problem**: Tests shouldn't require a model. But without tests, pipeline breaks silently.

**Decision**: "Mock" provider uses deterministic keyword matching (e.g., "kill" or "murder" → Violence event).

**Why**:
- Tests run in <1 second without GPU/CUDA
- CI doesn't require model infrastructure
- False negatives are acceptable (keyword miss) vs. false positives (hallucinated content)
- Validates the aggregation pipeline independently of model quality

**Trade-off**: Mock won't catch subtle content ("implied violence"). That's OK; real provider catches it.

---

## ADR-3: Local-First Privacy Model

**Decision**: Default provider is `llamacpp` (local); users *opt-in* to sending data to OpenAI.

**Why**:
- Book text is personal (reading history, interests)
- GDPR/privacy-conscious users need local option
- Builds trust ("I never saw your book" vs. "we sent it to the cloud")
- Feasible on consumer hardware (16GB laptop can run Qwen 4B)

**Trade-off**: Slower analysis (CPU inference ~2min per book vs. 30s cloud), but privacy is the point.

---

## ADR-4: Schema-Driven Extraction (Not Free-Form Text)

**Decision**: LLM outputs validated JSON schemas, not prose.

**Why**:
- Deterministic aggregation (can't sum up prose; can sum counts in JSON)
- Version stability (same schema version = comparable results)
- Auditable (JSON is data; can diff/track changes)
- Exportable (JSON → CSV/HTML trivially)

**Trade-off**: Requires careful prompt engineering to get LLM to output valid JSON reliably. Worth it for determinism.

---

## ADR-5: Evidence Traceability (Every Claim Has a Source)

**Decision**: Every content event includes chapter/location/excerpt, not just a category count.

**Why**:
- Parents can verify ("does that scene actually exist?")
- No black-box feeling ("why level 4?" → here's why)
- Accountability for rating system
- Catches LLM hallucinations ("that scene doesn't exist")

**Trade-off**: Storage bloat (storing excerpts vs. just counts). Storage is cheap; trust is expensive.

---

## ADR-6: Pluggable Providers (LLM Abstraction)

**Decision**: LLM logic is abstracted behind a provider interface. Three implementations: mock, llamacpp, openai_compatible.

**Why**: Models improve constantly. Without abstraction, upgrading the model means rewriting the pipeline. With abstraction, it's a config change.

**Trade-off**: Extra layer of indirection, but buys flexibility worth far more than the code cost.

---

## Summary: Guiding Principle

**Separation of Concerns**: Analysis logic ≠ LLM choice ≠ storage format

| Decision | Principle |
|----------|-----------|
| Chunking | Handle large books without context-limit pain |
| Mock provider | Test pipeline without models |
| Local-first | Privacy by default, opt-in to cloud |
| Schema-driven | Deterministic + auditable extraction |
| Evidence traces | Trust through transparency |
| Pluggable providers | Decouple core logic from LLM choice |

---

**Document Owner**: Architecture team  
**Last Updated**: July 2026
