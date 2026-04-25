# 🎓 Courseware Review (Beta)

> A Claude Code skill for **systematic, in-depth exam review** from lecture slides, PDFs, and textbooks. Rigorous, interactive, faithful to source material, with unlimited derivation depth.

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-orange.svg)](https://claude.ai/code)

---

## Overview

Courseware Review is a specialized Claude Code skill designed for students and learners who need to deeply understand course materials before exams. It transforms raw PDF courseware (slides, lecture notes, textbooks) into comprehensive, structured review documents through a multi-phase interactive process.

### Key Features

- **Faithful to Source Material**: Strictly follows the PDF's chapter order, derivation logic, and notation system. Never replaces original content with external knowledge.
- **Deep Derivations, Not Summaries**: Every algebraic transformation is written out in full intermediate steps — no "it's easy to see" shortcuts.
- **Interactive Learning Loop**: Outline confirmation → module-by-module teaching → active quizzes → doubt resolution → file commit → next module.
- **Three-Scale Auto-Pathing**: Automatically selects Fast (≤30 pages), Full (30–80 pages), or XL (>80 pages) path based on PDF size.
- **Image-Aware Processing**: Renders PDF pages as PNG for visual content analysis, with dynamic model-aware context budgets.
- **Self-Checking Output**: Every output document passes a rigorous self-inspection checklist with evidence commands.

---

## Quick Start

### Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed and authenticated
- The skill installed in your Claude Code skills directory
- Optional: `poppler` (for `pdftoppm` PDF-to-image rendering) — available via `conda install poppler` or your system package manager

### Installation

1. **Clone this repository:**

```bash
git clone https://github.com/yumenomajo/Courware-review.git
cd Courware-review
```

2. **Install the skill for Claude Code:**

Copy `SKILL.md` to your Claude Code skills directory:

```bash
# Linux / macOS
cp SKILL.md ~/.claude/skills/courseware-review-beta/SKILL.md

# Windows (PowerShell)
Copy-Item SKILL.md "$env:USERPROFILE\.claude\skills\courseware-review-beta\SKILL.md"
```

Or manually place `SKILL.md` at:
- **Linux/macOS**: `~/.claude/skills/courseware-review-beta/SKILL.md`
- **Windows**: `%USERPROFILE%\.claude\skills\courseware-review-beta\SKILL.md`

### Usage

Simply open Claude Code and trigger the skill with any of these phrases:

```
复习这份讲义
讲解这份课件
考试复习
review this courseware
exam review from PDF
```

Example workflow:

```
user: "/courseware-review-beta 读 slides/L5/TSA_Lecture5.pdf，输出到 L5/lec5.md"

Claude: PDF 25 pages, ~3 modules detected. Suggesting Fast path.
        Proceed to Phase 1: Roadmap?
```

---

## Architecture

### Phase 0: Quick Estimate

Zero-cost text extraction to estimate PDF scale and select the processing path. No images are loaded at this stage.

```
pdfinfo → total pages
pdf skill / pdftotext → full text extraction
grep for section titles → estimate module count K
→ Select Fast (≤30p) / Full (30–80p) / XL (>80p) path
```

### Phase 1: Roadmap

Creates a structured review outline with:
- YAML progress block for state tracking
- Table of contents with prerequisite dependencies
- Formula quick-reference table
- Image budget allocation per R8 rules
- Session recovery with consistency checks

### Phase 2: Module Cycle (Core Loop)

Each module goes through a 3-step cycle:

| Step | Name | Description |
|------|------|-------------|
| A | **READ** | Renders PDF pages, reads images with dynamic quotas, recovers session state |
| B | **EXPLAIN** | Full derivation with 5-section structure: logic, math, intuition, pitfalls, summary |
| C | **INTERACT + COMMIT** | Quizzes, doubt resolution, writes to file with MODULE_SUMMARY schema |

Three terminal states for the interaction phase:
- **A**: Quiz completed, no doubts
- **B**: User skipped quiz (tracked for review suggestion)
- **C**: Doubts raised and resolved with full derivations

### Phase 3: Final Review + Self-Check

After all modules complete:
- Cross-module synthesis and exam strategy guide
- 12-item self-inspection checklist with evidence commands
- Automatic cleanup of unreferenced intermediate PNG files

---

## Core Rules

| ID | Rule | Severity |
|----|------|----------|
| **R1** | No fabrication of values, tables, data points not present in PDF | 🔴 Critical |
| **R2** | Lock to original PDF notation — no equivalent substitutions | 🔴 Critical |
| **R3** | Visual-first processing for slides (PNG before text) | 🔴 Critical |
| **R4** | Single output directory for all files (md + PNG) | 🟡 Warning |
| **R5** | Image embeds limited to diagrams only, 0–2 per module | 🟡 Warning |
| **R6** | LaTeX `\|` escaping for all pipe characters in math | 🟡 Warning |
| **R7** | Tail-Read before Edit (last 200 lines or APPEND_HERE anchor) | 🟡 Warning |
| **R8** | Image flow control with model-aware context budgets (20MB limit) | 🔴 Critical |
| **R9** | Mandatory doubt cycle with 3 terminal states (A/B/C) | 🟡 Warning |
| **R10** | "Continue" must write file before next module | 🟡 Warning |
| **R11** | No length cap on mathematical derivations | 🔴 Critical |
| **R12** | Post-completion image cleanup (delete unreferenced PNGs) | 🟡 Warning |

---

## Output Format

Each output file includes:

```markdown
<!-- progress YAML block -->
# [Lecture Name] 复习路线图

## 目录总览          → Table of contents with difficulty ratings
## 公式速查表        → Quick-reference for all formulas

# Phase 2 — 逐步精讲

## 模块 N：[Name]
### 📖 逻辑复现      → Logical reconstruction with cross-references
### 🧮 数理推导      → Full derivation, no skipped steps
### 🧠 直觉与物理意义 → Intuition and practical meaning
### ⚠️ 陷阱与辨析    → Common pitfalls and confusions
### 📊 模块总结      → Three-column summary table

## 📌 模块 N 疑难补充 → Doubts resolved during interaction
<!-- MODULE_SUMMARY: 模块 N --> → Structured metadata block
<!-- END_SUMMARY -->
```

---

## Fallback Strategies

The skill is designed for Windows-first environments with graceful degradation:

| Issue | Fallback |
|-------|----------|
| No `pdftoppm` (Windows default) | `/pdf` skill text extraction → Python `fitz`/`pymupdf` → prompt user to install poppler |
| PDF encrypted | Inform user, request decrypted copy |
| Scanned PDF / OCR empty | Force PNG rendering at 200 DPI |
| Single-page formula unclear | Mark as `[原文]?`, ask user for confirmation |
| Large PNG (>4MB/page) | Reduce DPI 150→100, crop region, last resort: text extraction |
| Text garbled | Ignore text, fully image-based processing (R3) |
| Long module exceeds R8 quota | Batch read-write-release cycle, **no sampling** |

---

## Project Structure

```
Courware-review/
├── README.md              # This file
├── LICENSE                # MIT License
├── SKILL.md               # Claude Code skill definition
├── ARCHITECTURE.md        # Technical architecture details
├── CONTRIBUTING.md        # Contribution guidelines
└── examples/              # Example outputs (optional)
```

---

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Development Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/amazing-feature`)
3. Make your changes to `SKILL.md`
4. Test with a real PDF in Claude Code
5. Commit your changes (`git commit -m 'feat: add amazing feature'`)
6. Push to the branch (`git push origin feat/amazing-feature`)
7. Open a Pull Request

---

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

Built for the Claude Code ecosystem as a specialized skill for academic courseware review.
