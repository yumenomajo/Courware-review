# Architecture

Technical details of the Courseware Review skill's internal architecture and design decisions.

---

## Design Philosophy

The skill is built around a **four-phase pipeline** that progressively transforms raw PDF content into a structured, interactive review document. Each phase has a clear input, output, and termination condition.

```
Phase 0 (Estimate) → Phase 1 (Roadmap) → Phase 2 (Module Cycle) → Phase 3 (Finalize)
```

## Phase Details

### Phase 0: Quick Estimate

**Goal**: Determine PDF complexity without loading any images (zero API cost for image context).

**Tools used**:
- `pdfinfo` — extract page count
- `/pdf` skill or `pdftotext` — full text extraction
- `grep` — count section headings to estimate module count

**Output**: Path selection (Fast / Full / XL) based solely on page count.

**Why page count?** Avoids circular dependency — estimating module count from content would require reading content first, which defeats the purpose of a zero-cost estimate.

### Phase 1: Roadmap

**Goal**: Create a navigable outline that lets the user understand the scope before committing to the full review.

**Key mechanisms**:
- **Image budget allocation**: Calculates how many PNG images can be loaded per the R8 context budget, reserving capacity for the actual module content.
- **Cross-validation**: Reads ≥3 PNG images to verify that extracted text matches visual content. This catches PDFs where text extraction produces garbled output.
- **Session recovery**: On resume, reads all existing `MODULE_SUMMARY` blocks and performs a consistency check against the YAML progress block.

**Output format**: See `SKILL.md` → Output Format → Output template.

### Phase 2: Module Cycle

**The core loop.** Each module is processed through three sub-steps:

#### Step A: READ

- Renders the PDF pages for the current module using `pdftoppm` (or fallback)
- Reads PNG images with model-aware quotas (Sonnet: ≤5, Opus: ≤8, Haiku: ≤3 per message)
- For long modules exceeding the quota, implements a read-write-release cycle — images are consumed, the derivation is immediately written to the output file, then the context is freed by relying on the written file rather than in-memory images

#### Step B: EXPLAIN

Produces five structured subsections:

1. **📖 逻辑复现** — What does this module cover? How does it connect to the previous module?
2. **🧮 数理推导** — Full derivation with zero skipped steps. Every algebraic transformation shows intermediate states.
3. **🧠 直觉与物理意义** — Geometric/practical meaning in plain language.
4. **⚠️ 陷阱与辨析** — Common mistakes and conceptual confusions.
5. **📊 模块总结** — Three-column summary table (concept, content, formula).

The **Tail-Read mechanism (R7)** is critical here: instead of reading the entire file before each append, only the last 200 lines (or the `APPEND_HERE` anchor) is read. This keeps context usage bounded regardless of document length.

#### Step C: INTERACT + COMMIT

- Administers 2–3 progressive quiz questions (concept → application)
- Invites user doubts and resolves them with complete derivations
- Writes the resolved doubts and MODULE_SUMMARY to the output file
- Three terminal states: A (quiz passed), B (skipped), C (doubts resolved)

### Phase 3: Final Review

**Goal**: Polish the output document and verify quality.

1. Appends cross-module synthesis and exam strategy guide
2. Updates YAML progress block to `status: complete`
3. Runs a 12-item self-inspection checklist, each with an evidence command (not just checkbox ticking)
4. Cleans up unreferenced intermediate PNG files (R12)

---

## Key Data Structures

### YAML Progress Block

Embedded at the top of every output file as an HTML comment. Serves as the single source of truth for session state:

```yaml
status: in-progress          # planning | in-progress | complete
last_module: 2               # Last completed module number
modules_total: 5             # Total modules in this PDF
doubts_committed: true       # Whether the latest module's doubts were written
next_action: 开始模块 3       # Human-readable next step
updated: 2026-04-24          # ISO date of last write
```

### MODULE_SUMMARY Schema

Fixed-schema metadata block appended after each module's doubt resolution:

```
<!-- MODULE_SUMMARY: 模块 N -->
| 核心概念   | [list of concepts]  |
| 关键公式   | [formula names]     |
| 前置依赖   | [module/formula refs]|
| 后续用到   | [downstream refs]    |
| 用户疑点   | 🔴 X; 🟡 Y          |
| 终态类型   | A/B/C               |
<!-- END_SUMMARY -->
```

---

## Tail-Read / Append Mechanism (R7)

Traditional file append requires either:
- Reading the entire file (O(n) context cost)
- Appending blindly (risk of duplicate content or formatting errors)

The APPEND_HERE anchor pattern solves this:

1. On initial write: place `<!-- APPEND_HERE -->` at the end of the file
2. On subsequent writes: `Read` last 200 lines or `Grep` for the anchor
3. `Edit` the anchor → replace with new content + a fresh `<!-- APPEND_HERE -->`

This is O(1) context cost regardless of file size.

---

## Image Budget Management (R8)

The 20MB per-message API limit constrains how many PNG images can be loaded simultaneously. Budgets are model-dependent:

| Model | Per-message limit | Cumulative limit |
|-------|------------------|------------------|
| Sonnet (default) | ≤ 5 PNG | — |
| Opus 4.x (1M ctx) | ≤ 8 PNG | ≤ 30 PNG |
| Haiku | ≤ 3 PNG | — |

Budget allocation in Phase 1:
```
TOC_budget = min(actual TOC pages, 4)
sample_budget = min(K × 2, R8_quota - TOC_budget)
```

Long modules use a **read → write → release** cycle to stay within budget without losing content.

---

## Fallback Decision Tree

```
pdftoppm available?
├─ Yes → use pdftoppm for page rendering
└─ No
   ├─ /pdf skill available?
   │  ├─ Yes → use /pdf skill for text extraction
   │  └─ No
   │     ├─ Python fitz/pymupdf available?
   │     │  ├─ Yes → use fitz for key page rendering
   │     │  └─ No → prompt user to install poppler
```
