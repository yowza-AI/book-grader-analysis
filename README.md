# Book Grader: Evidence-Backed Content Analysis for Readers

> **Problem Solved**: Parents and readers need DETAILED content guidance for books, not vague ratings.

Current solutions (Goodreads, Common Sense Media, etc.) give you a number: "4 out of 5 stars for violence." That's useless.

**Book Grader solves this** by analyzing the actual text and producing reports like:

> **Violence: Level 4/5**
> - Chapter 4: Main character decapitates antagonist (graphically described, ~300 words)
> - Chapter 12: Street fight with blood/injury (moderate detail, ~150 words)
> - 7 other incidents of varying severity
>
> **What this means**: Violence is a significant component. Readers who find detailed combat scenes difficult should know this book isn't for them.

---

## Why This Approach

**Current rating sites fail because they:**
- Use crowd-sourced votes ("5 stars" from someone vs "1 star" from someone else—no consensus)
- Summarize instead of analyzing (miss context, nuance, accumulation)
- Never show their evidence ("why level 4?" → shrug)
- Don't account for FREQUENCY (one brutal scene vs. ten violent scenes are different)
- Treat all readers as the same age/sensitivity

**Book Grader's approach:**
- ✅ **Analyzes the actual text** (not summaries, not crowd opinion)
- ✅ **Shows evidence** (exact chapters, content snippets, line counts)
- ✅ **Counts patterns** (frequency matters; 50 swear words ≠ 2 slurs)
- ✅ **Runs locally** (your data stays private by default)
- ✅ **Deterministic & auditable** (same book = same analysis, every time)

---

## How It Works

```
Upload a book (.txt, .epub, .pdf)
    ↓
[Chunked Text Pipeline]
- Split into sized chunks (respects chapter boundaries)
- Never dumps 400K words into one LLM prompt
    ↓
[Content Extraction (local LLM or mock)]
- Scan each chunk for content events
- Extract: category, severity, evidence snippet, chapter/location
    ↓
[Aggregation & Counting]
- Deduplicate (same scene mentioned twice = one event)
- Count frequency per category
- Build severity profile
    ↓
[Report Generation]
- Age band recommendation (informal guidance, not official rating)
- Per-category breakdown (Violence: 4/5, Language: 2/5, etc.)
- Evidence summary with counts and locations
- Export as JSON/CSV/HTML/markdown
    ↓
[Output]
Parents/readers see:
- WHAT: specific content types and their severity
- WHERE: which chapters, how frequently
- CONTEXT: quoted evidence snippets
```

---

## Technical Approach: Pluggable Providers

The LLM layer is deliberately abstracted. Three providers available:

| Provider | Privacy | Speed | Quality | Hardware | Use Case |
|----------|---------|-------|---------|----------|----------|
| **mock** | 100% (offline keyword rules) | Fast | Basic | None | Dev/testing |
| **llamacpp** (local) | 100% (stays on your machine) | Slow (CPU) | Good | Laptop 16GB+ | Production, privacy-sensitive |
| **openai_compatible** | 0% (data leaves machine) | Fast | Excellent | Cloud | Analysis at scale |

One-line config change (`.env`), zero code changes. This is a design principle: **extract the uncertainty (LLM choice) from the core logic (evidence aggregation)**.

Why? Because:
- Model quality improves → just swap the model, not the pipeline
- Better hardware available → switch providers instantly
- Offline/online tradeoff → user's choice at runtime
- Testing → mock provider runs tests without any model

---

## Architecture Decisions

See [ARCHITECTURE_DECISIONS.md](ARCHITECTURE_DECISIONS.md) for detailed ADRs covering:

- **ADR-1**: Chunked text processing (not full-book prompts)
- **ADR-2**: Deterministic mock provider for testing
- **ADR-3**: Local-first privacy model
- **ADR-4**: Schema-driven extraction (JSON, not prose)
- **ADR-5**: Evidence traceability (every claim has proof)
- **ADR-6**: Pluggable providers

---

## Architecture: Layers

```
┌─────────────────────────────────────────────┐
│ Web UI (Browser)                            │
│ ├─ Upload form                              │
│ ├─ Progress streaming (live updates)        │
│ └─ Report viewer (JSON/CSV/HTML exports)    │
└──────────────┬──────────────────────────────┘
               ↓
┌─────────────────────────────────────────────┐
│ REST API (FastAPI)                          │
│ ├─ POST /analyze (chunked, streaming)       │
│ ├─ GET /report (retrieve analysis result)   │
│ ├─ DELETE /book (data removal)              │
│ └─ GET /models (available models + config)  │
└──────────────┬──────────────────────────────┘
               ↓
┌─────────────────────────────────────────────┐
│ Analysis Pipeline (Core Logic)              │
│ ├─ Text loader (txt/epub/pdf)               │
│ ├─ Chunker (respects chapters)              │
│ ├─ Extractor (LLM provider abstraction)     │
│ ├─ Aggregator (count, deduplicate, score)   │
│ └─ Reporter (age bands, formats)            │
└──────────────┬──────────────────────────────┘
               ↓
┌─────────────────────────────────────────────┐
│ Provider Layer (Pluggable)                  │
│ ├─ mock (keyword rules)                     │
│ ├─ llamacpp (local inference)               │
│ └─ openai_compatible (cloud)                │
└──────────────┬──────────────────────────────┘
               ↓
┌─────────────────────────────────────────────┐
│ Storage (SQLite + Local Filesystem)         │
│ ├─ Books metadata + analysis results        │
│ ├─ Evidence excerpts (for traceability)     │
│ └─ Audit trail (who ran what when)          │
└─────────────────────────────────────────────┘
```

---

## Development Approach


This matters for architect roles: shows how to direct AI tools for velocity without compromising architectural integrity. Every layer boundary, provider abstraction, and testing strategy reflects deliberate choices.

---


## Development Approach


This matters for architect roles: shows how to direct AI tools for velocity without compromising architectural integrity. Every layer boundary, provider abstraction, and testing strategy reflects deliberate choices.

---

**Status**: Production ready (Phase 1 complete)
**Hardware Tested**: Intel i9-13900H / 16GB RAM (CPU inference with Qwen 4B)
**Models Verified**: Qwen 3.5 4B-Q4_K_M (llama.cpp)
