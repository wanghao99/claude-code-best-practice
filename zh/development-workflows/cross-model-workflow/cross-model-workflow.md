# 跨模型（Claude Code + Codex）工作流

> 译自 [development-workflows/cross-model-workflow/cross-model-workflow.md](../../../development-workflows/cross-model-workflow/cross-model-workflow.md)，基于 2026-10-26 的 main 分支（commit 29f6dd2）。

基于 [claude-code-best-practice](https://github.com/shanraisshan/claude-code-best-practice) 和 [codex-cli-best-practice](https://github.com/shanraisshan/codex-cli-best-practice)

## 工作流

```
┌─────────────────────────────────────────────────────────────────────────┐
│              CROSS-MODEL CLAUDE CODE + CODEX WORKFLOW                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  STEP 1: PLAN                                          Claude Code      │
│  ─────────────                                         Opus 4.6         │
│  Open Claude Code in plan mode (Terminal 1).           Plan Mode        │
│  Claude interviews you via AskUserQuestion.                             │
│  Produces a phased plan with test gates.                                │
│                                                                         │
│  Output: plans/{feature-name}.md                                        │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 2: QA REVIEW                                     Codex CLI        │
│  ──────────────────                                    GPT-5.4          │
│  Open Codex CLI in another terminal (Terminal 2).                       │
│  Codex reviews plan against the actual codebase.                        │
│  Inserts intermediate phases ("Phase 2.5")                              │
│  with "Codex Finding" headings.                                         │
│  Adds to the plan — never rewrites original phases.                     │
│                                                                         │
│  Output: plans/{feature-name}.md (updated)                              │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 3: IMPLEMENT                                     Claude Code      │
│  ──────────────────                                    Opus 4.6         │
│  Start a new Claude Code session (Terminal 1).                          │
│  You implement phase-by-phase                                           │
│  with test gates at each phase.                                         │
│                                                                         │
│                              ▼                                          │
│                                                                         │
│  STEP 4: VERIFY                                        Codex CLI        │
│  ────────────────                                      GPT-5.4          │
│  Start a new Codex CLI session (Terminal 2).                            │
│  Codex verifies the implementation                                      │
│  against the plan.                                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## 跨模型工作流在生产中的实际形态

![Cross-Model Workflow](../../../development-workflows/cross-model-workflow/assets/cross-model-workflow.png)

*最后更新：2026-03-06*
