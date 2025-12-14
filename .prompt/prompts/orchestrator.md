# APE Prompt System Orchestrator

You are an orchestration assistant for the APE (Automated Prompt Engineering) system.

## Purpose

This file defines the **execution order** and **dependency relationships** between the prompt files in this system. Use this as the entry point when setting up or maintaining the prompt infrastructure.

## System Overview

```text
┌─────────────────────────────────────────────────────────────────────┐
│                    APE Prompt System Architecture                   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  PHASE 1: Repository Scan                                           │
│  ┌─────────────────────┐                                            │
│  │  repo-scanner.md    │──────┬──▶ .prompt/repo-summary.md         │
│  │  (meta prompt)      │      └──▶ .prompt/key-files-snapshot.md   │
│  └─────────────────────┘                                            │
│           │                                                         │
│           ▼                                                         │
│  PHASE 2: Prompt Generation                                         │
│  ┌─────────────────────────┐                                        │
│  │  meta-initial-prompt.md │──────┬──▶ .prompt/init.prompt.md      │
│  │  (meta prompt)          │      └──▶ .prompt/qa-deep.prompt.md   │
│  └─────────────────────────┘                                        │
│           │                                                         │
│           ▼                                                         │
│  PHASE 3: Prompt Optimization (Optional, Iterative)                 │
│  ┌──────────────────────────┐                                       │
│  │  meta-prompt-optimizer.md│──────▶ Revisions to init.prompt.md   │
│  │  (meta prompt)           │        and qa-deep.prompt.md          │
│  └──────────────────────────┘                                       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Execution Flow

### Phase 1: Repository Scanning

**Trigger**: Run when project is first set up, or when significant structural changes occur.

**Input**: Actual project source code and configuration files.

**Executor**: Any AI assistant (GitHub Copilot, Claude, etc.)

**Command**:

```text
READ .prompt/prompts/repo-scanner.md and execute its instructions.
```

**Outputs**:

- `.prompt/repo-summary.md` — Project metadata and structure overview
- `.prompt/key-files-snapshot.md` — Key file contents snapshot

**Validation**:

- [ ] `repo-summary.md` exists and is 80–150 lines
- [ ] `key-files-snapshot.md` exists and is 300–800 lines
- [ ] No placeholder text like "TODO" or "[FILL IN]"

---

### Phase 2: Prompt Generation

**Trigger**: Run after Phase 1 completes successfully.

**Input**:

- `.prompt/repo-summary.md` (as reference, NOT authoritative)
- `.prompt/key-files-snapshot.md` (as reference, NOT authoritative)
- **Actual source code** (AUTHORITATIVE)

**Executor**: Any AI assistant

**Command**:

```text
READ .prompt/prompts/meta-initial-prompt.md and execute its instructions.
```

**Outputs**:

- `.prompt/init.prompt.md` — Project onboarding and analysis prompt
- `.prompt/qa-deep.prompt.md` — Deep QA and risk analysis prompt

**Validation**:

- [ ] Both files have valid YAML frontmatter
- [ ] Both files contain model signature at the end
- [ ] `init.prompt.md` includes Source Code Authority Rule
- [ ] `qa-deep.prompt.md` includes Evidence Rule

---

### Phase 3: Prompt Optimization (Optional)

**Trigger**: Run to improve prompt quality. Can be run multiple times iteratively.

**Input**:

- `.prompt/init.prompt.md`
- `.prompt/qa-deep.prompt.md`

**Executor**: Ideally a **DIFFERENT** AI model than the one that generated the prompts (to avoid cognitive blind spots). If the same model is used, the optimizer will issue a warning.

**Command**:

```text
READ .prompt/prompts/meta-prompt-optimizer.md and execute its instructions.
```

**Outputs**:

- Revision suggestions for both prompt files

**Validation**:

- [ ] Revisions include concrete before/after comparisons
- [ ] Revisions address specific failure modes
- [ ] Optimizer model signature is recorded

---

## File Dependency Graph

```text
Source Code (authoritative)
    │
    ├──▶ repo-scanner.md
    │         │
    │         ├──▶ repo-summary.md (reference only)
    │         └──▶ key-files-snapshot.md (reference only)
    │
    ├──▶ meta-initial-prompt.md
    │         │
    │         ├──▶ init.prompt.md
    │         └──▶ qa-deep.prompt.md
    │
    └──▶ meta-prompt-optimizer.md
              │
              └──▶ (revisions to init.prompt.md, qa-deep.prompt.md)
```

## When to Re-run Each Phase

| Scenario | Phase 1 | Phase 2 | Phase 3 |
|----------|---------|---------|---------|
| Initial project setup | Yes | Yes | Optional |
| Major refactoring (new modules, renamed dirs) | Yes | Yes | Optional |
| Added/removed key dependencies | Yes | Optional | — |
| Prompt quality issues detected | — | — | Yes |
| Switching AI assistant | — | Optional | Yes |
| Minor code changes | — | — | — |

## Quick Start Commands

### Full Setup (All Phases)

```text
1. READ .prompt/prompts/repo-scanner.md and execute.
2. Verify outputs: .prompt/repo-summary.md, .prompt/key-files-snapshot.md
3. READ .prompt/prompts/meta-initial-prompt.md and execute.
4. Verify outputs: .prompt/init.prompt.md, .prompt/qa-deep.prompt.md
5. (Optional) READ .prompt/prompts/meta-prompt-optimizer.md and execute.
```

### Refresh Scan Only

```text
READ .prompt/prompts/repo-scanner.md and execute.
```

### Optimize Existing Prompts

```text
READ .prompt/prompts/meta-prompt-optimizer.md and execute.
```

## Troubleshooting

### "repo-summary.md not found" error

→ Run Phase 1 first.

### Prompts seem inaccurate or outdated

→ Re-run Phase 1, then Phase 2. The `.prompt/` reference files may be stale.

### Same issues keep appearing after optimization

→ Try using a different AI model for Phase 3. Self-optimization has cognitive blind spots.

### Generated content contradicts actual code

→ This is expected if Phase 1 outputs are stale. **Always trust source code over `.prompt/` files.**

---

## Constraints

- **Idempotency**: Running the same phase twice should produce consistent results.
- **Source Code Authority**: `.prompt/repo-summary.md` and `.prompt/key-files-snapshot.md` are **hints**, not facts. Always verify against actual source.
- **Model Diversity**: For best optimization results, use different models for generation (Phase 2) and optimization (Phase 3).
