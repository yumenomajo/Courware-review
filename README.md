# 🎓 Courseware Review (Beta)

> 一个 Claude Code skill，从 PDF 课件/讲义/教材中做**系统化、深度的考试复习**。忠于原文、交互式讲解、不跳步推导。

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-orange.svg)](https://claude.ai/code)

---

## 项目简介

Courseware Review 是一个专为考试复习设计的 Claude Code skill。它将原始 PDF 课件（幻灯片、讲义、教材）通过多阶段交互式流程，转化为结构化、深度拆解的复习文档。

### 核心特性

- **忠于原文**：严格跟随 PDF 的章节顺序、推导逻辑、记号系统。不用外部知识覆盖原文内容。
- **深度推导，不做摘要**：每一步代数变换都写出中间态，禁止"容易看出/如上所述/略"式的跳步。
- **交互式学习循环**：大纲确认 → 逐模块讲解 → 主动测验 → 答疑 → 写入文件 → 下一模块。
- **三档自动路径选择**：根据 PDF 页数自动选择 Fast（≤30页）、Full（30–80页）、XL（>80页）路径。
- **图像感知处理**：将 PDF 渲染为 PNG 进行视觉内容分析，按模型动态分配上下文预算。
- **输出自检**：每份输出文档通过 12 项自检清单，每项附带验证命令。

---

## 快速开始

### 前置要求

- 已安装并登录 [Claude Code](https://claude.ai/code) CLI
- 已将 skill 安装到 Claude Code 的 skills 目录
- 可选：`poppler`（用于 `pdftoppm` PDF 转图像渲染）— 通过 `conda install poppler` 或系统包管理器安装

### 安装

1. **克隆本仓库：**

```bash
git clone https://github.com/yumenomajo/Courware-review.git
cd Courware-review
```

2. **安装 skill 到 Claude Code：**

将 `SKILL.md` 复制到 Claude Code 的 skills 目录：

```bash
# Linux / macOS
mkdir -p ~/.claude/skills/courseware-review-beta
cp SKILL.md ~/.claude/skills/courseware-review-beta/SKILL.md

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\courseware-review-beta"
Copy-Item SKILL.md "$env:USERPROFILE\.claude\skills\courseware-review-beta\SKILL.md"
```

或直接手动放置：
- **Linux/macOS**: `~/.claude/skills/courseware-review-beta/SKILL.md`
- **Windows**: `%USERPROFILE%\.claude\skills\courseware-review-beta\SKILL.md`

### 使用

在 Claude Code 中用以下任意短语触发 skill：

```
复习这份讲义
讲解这份课件
考试复习
review this courseware
exam review from PDF
```

示例工作流：

```
user: "/courseware-review-beta 读 slides/L5/TSA_Lecture5.pdf，输出到 L5/lec5.md"

Claude: PDF 25 页，约 3 个模块，建议走 Fast 路径。进入 Phase 1: Roadmap？
```

---

## 架构流程

### Phase 0：快速估计

零成本文本提取，估算 PDF 规模并选择处理路径。此阶段不加载任何图像。

```
pdfinfo → 总页数
/pdf skill 或 pdftotext → 全文提取
grep 一级标题 → 估计模块数 K
→ 选择 Fast (≤30页) / Full (30–80页) / XL (>80页) 路径
```

### Phase 1：Roadmap

生成结构化复习大纲，包括：
- YAML 进度块（状态追踪）
- 带前置依赖的目录总览表
- 公式速查表
- 按 R8 规则分配图像预算
- 会话恢复时的一致性检查

### Phase 2：模块循环（核心）

每个模块经过 3 步循环：

| 步骤 | 名称 | 说明 |
|------|------|------|
| A | **READ 读取** | 渲染 PDF 页面、按动态配额读取图像、恢复会话状态 |
| B | **EXPLAIN 讲解** | 五段式讲解：逻辑复现、数理推导、直觉意义、陷阱辨析、模块总结 |
| C | **INTERACT + COMMIT 交互 + 写入** | 测验、答疑、写入 MODULE_SUMMARY |

三种终态：
- **A 类**：完成测验，无疑点
- **B 类**：用户跳过测验（标记建议回补）
- **C 类**：提出疑点并完成答疑

### Phase 3：终检 + 自检

所有模块完成后：
- 追加跨模块综合梳理与考试策略
- 执行 12 项自检清单，每项附验证命令
- 自动清理未被引用的中间 PNG 文件

---

## 核心规则

| 编号 | 规则 | 级别 |
|------|------|------|
| **R1** | **不臆造**：禁止编造 PDF 中不存在的数值、表格、示例 | 🔴 |
| **R2** | **记号锁定**：只用原文的记号，不做等价替换 | 🔴 |
| **R3** | **视觉优先**（幻灯片）：先看渲染后的 PNG，文字提取仅辅助 | 🔴 |
| **R4** | **单文件单目录**：所有输出放在同一个文件夹 | 🟡 |
| **R5** | **图像嵌入仅限 diagram**：每模块 0–2 张，非公式/表格 | 🟡 |
| **R6** | **LaTeX `\|` 转义**：数学公式中的竖线统一写成 `\|` | 🟡 |
| **R7** | **Tail-Read before Edit**：追加前优先读末尾 200 行或定位 APPEND_HERE 锚点 | 🟡 |
| **R8** | **图像流控**：按模型动态配额（Sonnet ≤5 张/消息，Opus ≤8 张/消息） | 🔴 |
| **R9** | **答疑循环必走**，三种终态（A/B/C） | 🟡 |
| **R10** | **"继续"≠跳过写文件**：先 COMMIT 再进下一模块 | 🟡 |
| **R11** | **推导无篇幅上限**：不砍内容，宁可分批多发消息 | 🔴 |
| **R12** | **完成后图像清理**：删除未被 markdown 引用的中间 PNG | 🟡 |

---

## 输出格式

每个输出文件包含：

```markdown
<!-- progress YAML 进度块 -->
# [课程名] 复习路线图

## 目录总览          → 带难度评级的模块列表
## 公式速查表        → 所有关键公式速查

# Phase 2 — 逐步精讲

## 模块 N：[名称]
### 📖 逻辑复现      → 与上模块衔接，依赖锚点链接
### 🧮 数理推导      → 逐行拆解，不跳步
### 🧠 直觉与物理意义 → 大白话解释
### ⚠️ 陷阱与辨析    → 常见错误、易混淆点
### 📊 模块总结      → 概念—内容—公式三列表

## 📌 模块 N 疑难补充 → 交互环节记录的疑点与解答
<!-- MODULE_SUMMARY: 模块 N --> → 结构化元数据块
<!-- END_SUMMARY -->
```

---

## 回退策略

Skill 默认面向 Windows 环境，支持逐级降级：

| 故障 | 回退方案 |
|------|----------|
| 缺 `pdftoppm`（Windows 默认） | `/pdf` skill 文本提取 → Python `fitz`/`pymupdf` → 提示装 poppler |
| PDF 加密 | 告知用户解密，或提供纯文本副本 |
| 扫描版 / OCR 为空 | 全页强制 PNG 渲染，DPI 提到 200 |
| 单页公式模糊 | 标注 `[原文]?`，问用户确认，不猜 |
| 渲染后 PNG 很大（>4MB/页） | DPI 150→100，或裁剪公式区域，最后才退化到文本提取 |
| 文本提取乱码 | 忽略文本，完全走图像路径（R3） |
| 长模块超 R8 配额 | 分批读-写-释放循环，**不采样** |

---

## 项目结构

```
Courware-review/
├── README.md              # 说明文档（中英双语）
├── LICENSE                # MIT 开源协议
├── SKILL.md               # Claude Code skill 核心定义
├── ARCHITECTURE.md        # 技术架构详解
├── CONTRIBUTING.md        # 贡献指南
└── examples/              # 示例输出（可选）
```

---

## 贡献

欢迎贡献！详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

### 开发流程

1. Fork 本仓库
2. 创建特性分支（`git checkout -b feat/your-feature-name`）
3. 修改 `SKILL.md`（核心逻辑）或文档文件
4. 用真实 PDF 在 Claude Code 中测试
5. 提交更改（`git commit -m 'feat: describe your change'`）
6. 推送分支（`git push origin feat/your-feature-name`）
7. 提交 Pull Request

---

## 许可证

本项目基于 MIT License 开源 — 详见 [LICENSE](LICENSE)。

---

## 致谢

为 Claude Code 生态构建的学术课件复习专用 skill。

---

<details>
<summary>🇬🇧 English Version (Click to expand)</summary>

## Overview

Courseware Review is a specialized Claude Code skill designed for students and learners who need to deeply understand course materials before exams. It transforms raw PDF courseware (slides, lecture notes, textbooks) into comprehensive, structured review documents through a multi-phase interactive process.

### Key Features

- **Faithful to Source Material**: Strictly follows the PDF's chapter order, derivation logic, and notation system. Never replaces original content with external knowledge.
- **Deep Derivations, Not Summaries**: Every algebraic transformation is written out in full intermediate steps — no "it's easy to see" shortcuts.
- **Interactive Learning Loop**: Outline confirmation → module-by-module teaching → active quizzes → doubt resolution → file commit → next module.
- **Three-Scale Auto-Pathing**: Automatically selects Fast (≤30 pages), Full (30–80 pages), or XL (>80 pages) path based on PDF size.
- **Image-Aware Processing**: Renders PDF pages as PNG for visual content analysis, with dynamic model-aware context budgets.
- **Self-Checking Output**: Every output document passes a rigorous self-inspection checklist with evidence commands.

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
mkdir -p ~/.claude/skills/courseware-review-beta
cp SKILL.md ~/.claude/skills/courseware-review-beta/SKILL.md

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\courseware-review-beta"
Copy-Item SKILL.md "$env:USERPROFILE\.claude\skills\courseware-review-beta\SKILL.md"
```

### Usage

Simply open Claude Code and trigger the skill with any of these phrases:

```
复习这份讲义
讲解这份课件
考试复习
review this courseware
exam review from PDF
```

## Architecture

### Phase 0: Quick Estimate

Zero-cost text extraction to estimate PDF scale and select the processing path. No images are loaded at this stage.

```
pdfinfo → total pages
/pdf skill or pdftotext → full text extraction
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

## Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

</details>
