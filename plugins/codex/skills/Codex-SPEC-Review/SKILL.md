---
name: Codex-SPEC-Review
description: Codex-first review workflow for OpenSpec or Spectra change plans, with grounded pre-implementation findings and a required output-style menu
user-invocable: true
---

# Codex SPEC Review

Use this skill when the user wants Codex to review an OpenSpec or Spectra change plan before implementation.

This skill is for plan review, not implementation. The goal is to identify problems that should be resolved before coding starts, grounded in the actual change artifacts and any referenced repository files.

## What this skill does

It performs a silent pre-implementation review across three dimensions:
1. logical consistency
2. execution risk
3. execution-detail completeness

Then it pauses and asks the user which output style they want.

## Expected input

The user will usually provide a command-like request containing a change name after `$spec-double-review`.

Examples:
- `$spec-double-review "2026-04-30-speak-graph-segment-driven"`
- `$spec-double-review "a2f2-mouth-tuner"`

The quoted value is the target change slug.

If no explicit slug is provided:
- first infer it from nearby conversation context if the target is unambiguous
- otherwise list the available changes and ask the user to choose one

## Operating rules

- Stay read-only.
- Do not modify files, generate patches, or propose code edits unless the user separately asks for implementation help later.
- Ground every finding in the actual plan files and, when relevant, in verified repository paths or symbols.
- Do not expose the internal review process. Only show the required menu first, then the final conclusions after the user chooses a style.
- Do not skip the style-selection wait.

## Phase 1: Locate and load the change

### 1. Parse the change name

Support both of these forms:
- full slug, for example `2026-04-30-speak-graph-segment-driven`
- short slug tail, for example `speak-graph-segment-driven`

### 2. Resolve the matching directory

Search in this order:
1. `openspec/changes/<slug>/`
2. `openspec/changes/archive/<slug>/`
3. if no exact match exists, fuzzy-match the slug tail across both locations

If nothing matches:
- list available changes
- stop there
- ask the user to pick one

### 3. Read all relevant artifacts

Load every applicable artifact under the selected change:
- `proposal.md` — required
- `design.md` — if present
- `tasks.md` — if present
- `specs/*/spec.md` — if present

If `tasks.md`, `design.md`, or a spec references files, classes, functions, modules, directories, or symbols in the repository, verify they actually exist before relying on them.

If a referenced file or symbol cannot be found, treat that as a review finding when it materially affects implementation clarity.

## Phase 2: Silent three-angle review

Perform this analysis silently. Do not show interim reasoning.

### A. Logic consistency

Look for:
- mismatch between stated goals and described design
- hidden assumptions that are required but not stated
- missing edge cases such as empty input, failure modes, retries, concurrency, rollback, or partial-completion behavior
- tasks that do not fully implement the proposal
- inconsistent terminology for the same concept
- requirements stated in one artifact but omitted or contradicted in another

### B. Risk assessment

Look for:
- technical uncertainty around APIs, frameworks, libraries, protocols, or external systems
- compatibility risks with the runtime, host environment, or existing behavior
- integration risks across related subsystems, tools, or data flows
- unclear migration, rollout, rollback, or failure-containment strategy
- weak or missing validation and test coverage plans

### C. Execution-detail quality

Look for:
- vague instructions such as “appropriately”, “as needed”, “if necessary”, or “handle carefully”
- missing acceptance criteria or unclear done conditions
- incorrect, ambiguous, or nonexistent file/class/function references
- missing prerequisites, ordering constraints, or handoff details
- steps that are too high-level for a different engineer to execute reliably

## Severity guidance

Prioritize findings that should block implementation from starting:
- logic contradictions that undermine the design
- high-risk unknowns that could cause rework or breakage
- missing execution detail that makes the plan non-actionable

Do not pad the answer with minor nits. Prefer a shorter, sharper set of high-value findings.

## Phase 3: Required user menu

After the review is complete, do not present findings yet.

Show exactly this menu and wait for the user to reply:

```text
✅ 審查完成！請選擇輸出風格：

1. 專業說明　　— 技術精確用語，適合工程師直接對應程式碼
2. 小學生版　　— 用生活比喻解釋，任何人都能看懂問題在哪
3. 兩種都要　　— 先專業，再小學生版

請回覆 1、2 或 3：
```

Do not assume a default. Do not emit findings before the user answers.

## Phase 4: Final output contract

Only include these two sections in the final answer:
- problems that must be resolved before implementation starts
- overall assessment

Do not include a full analysis table, chain-of-thought, or extra sections.

### Style 1 — Professional

Use technically precise language.

Output shape:

```markdown
## Spec Review: <change-name>

### 需要在開始實作前解決的問題

1. **[問題標題]**
   - 性質：邏輯矛盾 / 高風險 / 執行細節不足
   - 說明：<具體描述，附文件位置 proposal.md / design.md / tasks.md / specs/...>
   - 建議：<明確的修正方向>

（若無問題，標記「未發現需阻擋實作的問題」）

---

### 整體評估

**結論**：可執行 / 需要修正後再執行 / 建議重新設計

**理由**：<一到兩句說明評估依據>
```

### Style 2 — Elementary-school version

Use plain everyday analogies and avoid technical jargon. Explain each issue in a way that a non-technical reader could follow.

Output shape:

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

Output the full Style 1 result first.
Then output a separator line.
Then output the full Style 2 result.

## Review-quality requirements

- Prefer evidence over confidence.
- If a point is uncertain, label it clearly instead of overstating it.
- Cite the exact artifact location for every material finding.
- When repository verification was needed, mention the verified path or missing reference only inside the finding itself.
- Focus on issues that matter before implementation starts, not hypothetical post-implementation refinements.

## Guardrails

- Read-only only.
- No file writes.
- No code changes.
- No hidden implementation planning presented as review output.
- No final conclusions before the user selects an output style.
- Final output must contain only the required problem list and the overall assessment.
