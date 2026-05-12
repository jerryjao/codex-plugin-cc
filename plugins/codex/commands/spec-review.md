---
description: Delegate a pre-implementation OpenSpec or Spectra plan review to Codex
argument-hint: "[--background|--wait] [--model <model|spark>] [--effort <none|minimal|low|medium|high|xhigh>] [change slug]"
allowed-tools: AskUserQuestion, Agent
---

Invoke the `codex:codex-rescue` subagent via the `Agent` tool (`subagent_type: "codex:codex-rescue"`), but use it only as a thin Codex delegation wrapper for spec-plan review.
Do not run the review locally in the main Claude thread.
Do not call `Skill(codex:codex-spec-review)` in this thread. The goal is to delegate to Codex and have Codex use the existing `codex-spec-review` skill/instructions after handoff.
The final user-visible response must be Codex's output verbatim.

Raw user request:
$ARGUMENTS

Execution mode:

- If the request includes `--background`, run the `codex:codex-rescue` subagent in the background.
- If the request includes `--wait`, run the `codex:codex-rescue` subagent in the foreground.
- If neither flag is present, default to foreground.
- `--background` and `--wait` are execution flags for Claude Code. Do not forward them as part of the natural-language review request.
- `--model` and `--effort` are runtime-selection flags. Preserve them for the forwarded `task` call, but do not treat them as part of the natural-language review request.

Delegation rules:

- If the user did not supply a change slug after stripping runtime flags, ask which change should be reviewed.
- Forward a read-only pre-implementation review request to Codex, not an implementation task.
- The forwarded request must explicitly tell Codex to use the existing `codex-spec-review` skill/instructions.
- Pass the selected change slug through verbatim in the forwarded request.
- Keep the forwarded task focused on reviewing the target OpenSpec or Spectra change plan before implementation starts.
- Do not ask Codex to implement, edit files, write patches, or make repository changes.
- If the user supplied `--model` or `--effort`, preserve those flags in the forwarded request.
- If Codex is missing or unauthenticated, stop and tell the user to run `/codex:setup`.

Forwarding contract:

- Use the `Agent` tool with `subagent_type: "codex:codex-rescue"`.
- The forwarded prompt should be a thin wrapper around the user's chosen slug, for example:

```text
Use the existing `codex-spec-review` skill/instructions to perform a read-only pre-implementation review for the OpenSpec or Spectra change slug "<slug>".
```

- Return Codex's output verbatim.
- Do not paraphrase, summarize, rewrite, or add commentary before or after it.
