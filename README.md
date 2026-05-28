# Claude Code to Figma

[![Made for Claude Code](https://img.shields.io/badge/Made%20for-Claude%20Code-blueviolet?style=flat-square&logo=anthropic)](https://docs.anthropic.com/en/docs/claude-code)
[![Figma MCP](https://img.shields.io/badge/Figma-MCP%20Server-ff7262?style=flat-square&logo=figma)](https://www.npmjs.com/package/@anthropic-ai/figma-mcp)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

Design System compliance for AI-generated Figma designs. 4 skills, 3-step preflight, zero raw values.

> **Quick start:** Clone, copy skills to `.claude/skills/`, paste your Figma URL into `CLAUDE.md`, say "let's start". [Full install guide below.](#installation)

[Chinese / 中文版](README.zh-CN.md)

---

## Why?

AI can write to Figma now. But without guidance, it builds everything from scratch — hardcoded hex colors, arbitrary font sizes, raw spacing values. The result looks right but is completely disconnected from your Design System. Every color is a magic number. Every component is a one-off. Your design tokens might as well not exist.

cc2figma fixes this with 4 Claude Code Skills that enforce Design System compliance at every step:

- Components are Instances of your Master Components, not rebuilt from scratch
- Colors, fonts, spacing, and radii bind to Variables and Styles, not raw values
- Every write to Figma is automatically verified for token compliance

---

## Before & After

![With Skills vs Without Skills](assets/before-after.png)

> **With Skills:** Master Component Instances, all visual values bound to Design System Variables and Styles.
> **Without Skills:** Hardcoded colors, arbitrary spacing, components built from scratch — looks right, but zero Design System connection.

---

## Preflight System

Before every design session, a 3-step parallel check ensures everything is connected:

![Preflight Status](assets/preflight-status.png)

MCP connection, file permissions, libraries, styles, variables, and components — all verified in parallel before a single node is created.

---

## Usage Examples

Say "let's start" to run preflight, then describe what you want:

    You: Build a login page with email and password fields
    Claude: [searches DS for Form, Input, Button components]
            [creates Instances, binds all tokens]
            [takes screenshot for verification]

    You: Add a registration form next to it with a confirm password field
    Claude: [reuses same DS components, adds new section]
            [QA verifies all bindings automatically]

    You: Here's a screenshot of the dashboard we need
    Claude: [analyzes reference, outputs structured Design Brief]
            [builds section by section, verifying after each step]

Every interaction follows the same loop: **search DS** → **create Instances** → **bind tokens** → **verify**.

---

## What's Included

### 4 Skills

| Skill | Trigger | What it does |
| ----- | ------- | ------------ |
| `figma-preflight` | "let's start", first Figma URL | 3-step parallel check + Token Map + Component Registry |
| `component-rules` | Any UI construction task | Library-first lookup, Auto Layout, semantic naming |
| `figma-style-binding` | Any color, font, or spacing operation | Enforce Variable / Style binding + post-write QA verification |
| `reference-interpreter` | Share screenshot, URL, or description | Output structured Design Brief before building |

### How They Work Together

    "let's start"
        |
        v
    ① preflight ─── one-time setup, load Token Map + Component Registry
        |
        v
    ② reference-interpreter ─── optional: screenshot / reference → Design Brief → user confirms
        |
        v
    ┌→ ③ component-rules ─── search Library, use Instances
    │       |
    │  ④ figma-style-binding ─── bind every value to a token, then auto-verify
    │       |
    └── next section (loop until done)

---

## Good For / Not For

| Good For | Not For |
| -------- | ------- |
| Building pages from an existing Design System | Creating a Design System from scratch |
| Describing UI in natural language, Claude builds it in Figma | Pixel-perfect illustration or icon drawing |
| Working in a DS file or a file with a linked DS library | Free-form design without a Design System |
| Ensuring 100% token compliance in design output | FigJam / whiteboard workflows |

---

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI / Desktop / VS Code)
- [Figma MCP Server](https://www.npmjs.com/package/@anthropic-ai/figma-mcp) installed and authenticated
- A Figma file with a Design System (locally defined or linked via Library)

### Install

```bash
# 1. Clone the repo
git clone https://github.com/senlindesign/cc2figma.git

# 2. Copy skills to your project
cp -r cc2figma/.claude/skills/* your-project/.claude/skills/

# 3. Copy config
cp cc2figma/.claude/settings.json your-project/.claude/settings.json

# 4. Copy CLAUDE.md template
cp cc2figma/CLAUDE.md.template your-project/CLAUDE.md
```

### Configure CLAUDE.md

Open `CLAUDE.md` and paste your Figma file URL:

```markdown
# Figma Design Project

- **Figma file:** <https://www.figma.com/design/YOUR_FILE_KEY/...>
- **Fonts:** [leave blank — Preflight auto-detects from your DS]
- **Session goal:** [what are we designing today?]

## Rules

1. Every visual value must bind to a Style or Variable.
2. Always search connected libraries before building any component from scratch.
3. Never start designing before the Design Brief is confirmed.
```

Then say "let's start" in Claude Code. Preflight handles the rest.

---

## Supported Scenarios

| Scenario | How It Works |
| -------- | ------------ |
| **Working in a DS file directly** | Preflight reads all local Styles, Variables, and Components. Full token binding. |
| **New file with a linked DS library** | Component Instances inherit all token bindings from masters automatically. Library variables can be imported for new frames. |

---

## Directory Structure

```
your-project/
├── CLAUDE.md                          # Project config (Figma URL, fonts, rules)
└── .claude/
    ├── settings.json                  # Permissions + QA Hook
    └── skills/
        ├── figma-preflight/           # 3-step parallel check + Token Map + Component Registry
        ├── component-rules/           # Library-first, Auto Layout, naming
        ├── figma-style-binding/       # Color / Text / Spacing binding + QA verification
        └── reference-interpreter/     # Reference → Design Brief
```

---

## Known Limitations

| Limitation | Workaround |
| ---------- | ---------- |
| Doubly-nested Instances can't be directly modified | `detachInstance()` on the outer instance first |
| `getLocalVariablesAsync()` can't see library variables | Instances inherit tokens automatically; use `importVariableByKeyAsync` for new frames |
| Invalid variant values in `setProperties` roll back the entire script | Read `componentPropertyDefinitions` to confirm valid values first |

---

## Contributing

Issues and PRs are welcome. If you have a Figma design workflow that could benefit from a new Skill, describe it in an Issue.

---

MIT © 2025 Sen Lin

---

## CIRCL Fork Notes (circl-tyo/claude2figma)

このリポは upstream `senlindesign/claude2figma` (MIT) の CIRCL 用 fork。

### 構造方針

- `.claude/skills/` 配下は upstream 由来。**直接編集しない** (リベースで破壊される)
- CIRCL カスタムの skill 本体は本 repo 外: `~/.claude/skills/circl-figma-{preflight,component-rules,style-binding,reference-interpreter}/`
- upstream と CIRCL skill は別物として共存させる (同名 conflict を避けるため、CIRCL 側は `circl-figma-` prefix)

### 上流追従ポリシー (月次)

毎月初週に以下を実施:

```bash
cd ~/github/circl/agents/claude2figma
git fetch upstream
git log upstream/main --oneline -- .claude/skills/   # skill 配下の差分
git merge upstream/main                                # or rebase
```

判断軸:

- upstream の core ロジック (Auto Layout / Binding Hierarchy / QA-1) 改修 → 取り込み、`~/.claude/skills/circl-figma-*/SKILL.md` に反映
- upstream の UX 文言改修 → 取り込まない (CIRCL は日本語 / output-style 準拠)
- upstream の新 skill 追加 → 個別評価。要件があれば `circl-figma-` prefix で新規作成

汎用的な fix (typo / Figma plugin API の誤り) は upstream に PR する。CIRCL 固有カスタム (Tokens Allowlist / prh / page-naming 連携) は upstream に PR しない。

### CIRCL カスタム (差分要点)

| 変更点 | 場所 | 理由 |
|---|---|---|
| skill name prefix `circl-figma-` | `~/.claude/skills/circl-figma-*` | CIRCL 命名規則統一 |
| Tokens Allowlist 参照を強制 | `circl-figma-component-rules` Rule 5, `circl-figma-style-binding` | M2 Approved 35 件以外を bind しないため |
| prh 辞書 (textlint MCP) 連携 | `circl-figma-component-rules` Rule 6, `circl-figma-style-binding` QA-2 | M3 構築の CIRCL 共通 + MOOV 28 ルールで表記揺れ防止 |
| Component Registry 連携を disabled 化 | `circl-figma-preflight` | C2 タスクで構築予定。それまでは skip |
| Opacity の bind 規則 | `circl-figma-style-binding` | M2 で `opacity/disabled` / `opacity/overlay` 追加のため |
| Page 命名規則整合 (`figma-page-naming`) | `circl-figma-component-rules` Rule 7 | 既存 CIRCL skill との接続 |

ライセンスは upstream の MIT を継承 (`LICENSE` ファイル維持)。
