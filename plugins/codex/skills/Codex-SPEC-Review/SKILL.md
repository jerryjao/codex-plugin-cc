---
name: Codex-SPEC-Review
description: Review an OpenSpec/Spectra change plan with deep logic analysis, risk assessment, and execution-detail checks before implementation
user-invocable: true
---

# Codex SPEC Review

Use this skill to review a Spectra or OpenSpec change plan before implementation. The goal is to identify blocking logic gaps, execution risks, and missing details, then wait for the user to choose how the findings should be presented.

## Input

The change name appears after `$spec-double-review`, usually in quotes.

Examples:
- `$spec-double-review "2026-04-30-speak-graph-segment-driven"`
- `$spec-double-review "a2f2-mouth-tuner"`

If the user does not provide a name:
- infer it from nearby conversation context when possible
- otherwise ask the user which change to review

## Phase 1: Locate and load the change plan

### 1. Parse the change name

Support both:
- full slug, like `2026-04-30-speak-graph-segment-driven`
- short slug tail, like `speak-graph-segment-driven`

### 2. Find the matching directory

Search in this order:
1. `openspec/changes/<slug>/`
2. `openspec/changes/archive/<slug>/`
3. if exact match fails, fuzzy-match by slug tail in both locations

If no match exists, stop and ask the user to choose a valid change after listing available changes.

### 3. Read all relevant artifacts

Read every applicable file under the chosen change:
- `proposal.md` — required
- `design.md` — if present
- `tasks.md` — if present
- `specs/*/spec.md` — if present

If `tasks.md` references files, classes, functions, or symbols, verify that they actually exist in the repository before relying on them.

## Phase 2: Silent three-angle review

Do not show this analysis process to the user. Perform it internally.

### A. Logic consistency

Check for:
- mismatch between goals and design
- hidden assumptions
- missing boundary conditions such as error paths, empty values, or concurrency
- tasks that do not fully deliver the proposal
- inconsistent terminology for the same concept

### B. Risk assessment

Check for:
- technical uncertainty in APIs, frameworks, or external systems
- compatibility risks with runtime or host environment constraints
- integration risks across related subsystems
- weak or missing validation and test strategy

### C. Execution detail quality

Check for:
- vague instructions such as “appropriately” or “if needed”
- missing acceptance criteria
- incorrect or nonexistent file, class, or function references
- omitted prerequisite steps

## Phase 3: Mandatory style-selection menu

After the review is complete, do not present findings immediately. First show exactly this menu and wait for the user to answer:

```text
✅ 審查完成！請選擇輸出風格：

1. 專業說明　　— 技術精確用語，適合工程師直接對應程式碼
2. 小學生版　　— 用生活比喻解釋，任何人都能看懂問題在哪
3. 兩種都要　　— 先專業，再小學生版

請回覆 1、2 或 3：
```

Do not skip the wait. Do not assume a default style.

## Phase 4: Final output contract

Only output these two sections:
- problems that must be resolved before implementation starts
- overall assessment

### Style 1 — Professional

Format:

```markdown
## Spec Review: <change-name>

### 需要在開始實作前解決的問題

1. **[問題標題]**
   - 性質：邏輯矛盾 / 高風險 / 執行細節不足
   - 說明：<具體描述，附文件位置 proposal.md / design.md / tasks.md>
   - 建議：<明確的修正方向>

（若無問題，標記「未發現需阻擋實作的問題」）

---

### 整體評估

**結論**：可執行 / 需要修正後再執行 / 建議重新設計

**理由**：<一到兩句說明評估依據>
```

### Style 2 — Elementary-school version

Use everyday analogies and avoid jargon.

Format:

```markdown
## 計畫健檢報告：<change-name>

### 開工前要先解決的問題

1. **[問題標題（比喻化）]**
   - 這是什麼問題：<用小學生能懂的比喻說明>
   - 為什麼要先解決：<說明若不解決會發生什麼，用日常情境比喻>
   - 怎麼解決：<簡單的行動方向>

（若無問題，說「目前沒有需要先解決的問題，可以放心開工！」）

---

### 最終結論

**可以開工嗎？**：可以！/ 先修一下再開工 / 這份計畫需要重新想一想

**原因**：<用比喻說明，一到兩句>
```

### Style 3 — Both

Output the full Style 1 result first, then a separator line, then the full Style 2 result.

## Guardrails

- Read-only only: never write, edit, or modify repository files as part of the review itself.
- Every finding must be grounded in specific document locations.
- Keep phases 1 and 2 silent.
- Always show the Phase 3 menu before any conclusions.
- Final output must contain only the problem list and the overall assessment.
