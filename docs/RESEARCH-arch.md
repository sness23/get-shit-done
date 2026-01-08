# GSD Architecture Deep Dive

> A comprehensive technical analysis of the Get Shit Done (GSD) meta-prompting and context engineering system for Claude Code.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Core Philosophy](#core-philosophy)
3. [System Architecture](#system-architecture)
4. [Installation System](#installation-system)
5. [Command System](#command-system)
6. [Template System](#template-system)
7. [Workflow Engine](#workflow-engine)
8. [Reference Documents](#reference-documents)
9. [Context Engineering](#context-engineering)
10. [Configuration System](#configuration-system)
11. [Git Integration Strategy](#git-integration-strategy)
12. [Subagent Architecture](#subagent-architecture)
13. [Key Design Patterns](#key-design-patterns)
14. [Complete File Hierarchy](#complete-file-hierarchy)
15. [Project Lifecycle Example](#project-lifecycle-example)

---

## Executive Summary

GSD (Get Shit Done) is a meta-prompting and context engineering system that transforms Claude Code into a structured project execution engine. Rather than treating Claude as a simple code assistant, GSD provides:

- **18 slash commands** for project lifecycle management
- **13 workflow definitions** for multi-step procedures
- **14+ templates** for consistent artifact generation
- **8 reference documents** for quality and consistency
- **Smart context assembly** via `@file` syntax

**Key Metrics:**
- Current version: 1.3.26
- NPM package: `get-shit-done-cc`
- Target audience: Solo developers using Claude Code as primary implementer

---

## Core Philosophy

GSD operates on principles that explicitly reject enterprise development patterns:

### 1. Solo Developer Model
- No teams, stakeholders, sprints, or story points
- Claude is the builder; user is the visionary
- Effort estimated in Claude execution time, not human hours

### 2. Plans Are Prompts
- PLAN.md files ARE executable prompts
- No transformation step between plan and execution
- XML structure provides precise instructions

### 3. Scope Control Through Atomicity
- 2-3 tasks per plan maximum
- Each plan independently executable
- Quality degradation curve: 0-30% context (peak) → 70%+ (poor)

### 4. Claude Automates Everything
- If it has CLI/API, Claude does it
- Checkpoints only for verification/decisions
- Never for deployments, builds, file operations

### 5. Anti-Enterprise Stance
- No team structures or stakeholder management
- No human hour estimates
- No change management theater
- Delete corporate PM patterns

---

## System Architecture

### High-Level Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER INTERACTION                             │
├─────────────────────────────────────────────────────────────────────┤
│  /gsd:new-project → /gsd:create-roadmap → /gsd:plan-phase          │
│                         ↓                        ↓                   │
│                    ROADMAP.md              PLAN.md                   │
│                         ↓                        ↓                   │
│                    STATE.md ←──── /gsd:execute-plan                 │
│                         ↓                        ↓                   │
│                   SUMMARY.md ← ─ ─ ─ ─ ─ ─ ─ ─ ─┘                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Relationships

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│    COMMANDS      │────▶│    WORKFLOWS     │────▶│    TEMPLATES     │
│  (User Entry)    │     │  (Procedures)    │     │  (Structures)    │
│  18 .md files    │     │  13 .md files    │     │  14+ .md files   │
└────────┬─────────┘     └────────┬─────────┘     └──────────────────┘
         │                        │                        ▲
         │                        │                        │
         ▼                        ▼                        │
┌──────────────────┐     ┌──────────────────┐              │
│   REFERENCES     │     │   ARTIFACTS      │──────────────┘
│   (Guidance)     │     │  (Generated)     │
│   8 .md files    │     │  PROJECT/STATE   │
└──────────────────┘     │  ROADMAP/PLAN    │
                         │  SUMMARY/etc     │
                         └──────────────────┘
```

---

## Installation System

### Installer: `bin/install.js`

The installer handles both interactive and non-interactive installation:

```bash
npx get-shit-done-cc          # Interactive prompt
npx get-shit-done-cc --global # Install to ~/.claude/
npx get-shit-done-cc --local  # Install to ./.claude/
```

### Path Replacement Logic

The critical installation feature is **path replacement**. During copy:

```javascript
// For .md files only:
// Global: ~/.claude/ or $CLAUDE_CONFIG_DIR
// Local:  ./.claude/

content.replace(/~\/\.claude\//g, pathPrefix)
```

This allows templates to reference other files via `@~/.claude/...` syntax, which gets rewritten based on installation location.

### Installation Output Structure

```
{dest}/
├── commands/gsd/           # 18 command files
│   ├── new-project.md
│   ├── create-roadmap.md
│   └── ...
└── get-shit-done/
    ├── references/         # 8 guidance files
    ├── templates/          # 14+ template files
    └── workflows/          # 13 workflow files
```

---

## Command System

### Command File Format

Each command follows a strict structure:

```markdown
---
description: "Short description for /help"
argument-hint: "[optional-arg]"
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
What this command accomplishes.
</objective>

<execution_context>
@~/.claude/get-shit-done/references/some-ref.md
@~/.claude/get-shit-done/templates/some-template.md
</execution_context>

<process>
<step name="step-name">
Instructions for execution.
</step>
</process>

<success_criteria>
- [ ] Criterion 1
- [ ] Criterion 2
</success_criteria>
```

### Complete Command Reference

| Command | Purpose | Creates |
|---------|---------|---------|
| `/gsd:new-project` | Initialize with deep context gathering | PROJECT.md, config.json |
| `/gsd:create-roadmap` | Break project into phases | ROADMAP.md, STATE.md, phase dirs |
| `/gsd:map-codebase` | Analyze existing codebase | codebase/ (7 docs) |
| `/gsd:plan-phase [N]` | Create execution plan | XX-YY-PLAN.md |
| `/gsd:execute-plan [path]` | Execute PLAN.md | XX-YY-SUMMARY.md |
| `/gsd:progress` | Check status | (display) |
| `/gsd:discuss-phase [N]` | Gather context | CONTEXT.md |
| `/gsd:research-phase [N]` | Deep research | RESEARCH.md |
| `/gsd:list-phase-assumptions [N]` | Preview approach | (display) |
| `/gsd:add-phase [desc]` | Append phase | ROADMAP.md |
| `/gsd:insert-phase [N] [desc]` | Insert decimal phase | ROADMAP.md |
| `/gsd:new-milestone [name]` | Create milestone | ROADMAP.md, STATE.md |
| `/gsd:complete-milestone [ver]` | Archive milestone | MILESTONES.md, git tag |
| `/gsd:discuss-milestone` | Plan next milestone | (display) |
| `/gsd:pause-work` | Create handoff | .continue-here, STATE.md |
| `/gsd:resume-work` | Restore session | (display) |
| `/gsd:consider-issues` | Review issues | ISSUES.md |
| `/gsd:help` | Display usage | (display) |

---

## Template System

Templates define artifact structure. They're **living documents** that evolve as projects progress.

### Core Templates

| Template | Location | Purpose |
|----------|----------|---------|
| project.md | `.planning/PROJECT.md` | Vision, requirements, constraints |
| state.md | `.planning/STATE.md` | Current position, decisions, blockers |
| roadmap.md | `.planning/ROADMAP.md` | Phase breakdown, progress tracking |
| phase-prompt.md | `.planning/phases/XX-YY-PLAN.md` | Executable plan structure |
| summary.md | `.planning/phases/XX-YY-SUMMARY.md` | Execution results |
| config.json | `.planning/config.json` | Workflow configuration |

### Codebase Map Templates

For brownfield projects, `/gsd:map-codebase` creates:

```
.planning/codebase/
├── STACK.md          # Languages, frameworks, dependencies
├── ARCHITECTURE.md   # Patterns, layers, data flow
├── STRUCTURE.md      # Directory layout
├── CONVENTIONS.md    # Code style, naming
├── TESTING.md        # Test framework, patterns
├── INTEGRATIONS.md   # External services, APIs
└── CONCERNS.md       # Tech debt, known issues
```

### Template Evolution

Templates update throughout the project lifecycle:

- **PROJECT.md**: Requirements validated/invalidated, decisions logged
- **STATE.md**: Position after each plan, new decisions, blockers
- **ROADMAP.md**: Phase completion, new phases, milestone shipping

---

## Workflow Engine

Workflows are multi-step procedures that commands orchestrate.

### Available Workflows

| Workflow | Purpose |
|----------|---------|
| plan-phase.md | Create detailed task breakdown |
| execute-phase.md | Run PLAN.md, create SUMMARY.md |
| create-roadmap.md | Break project into phases |
| create-milestone.md | Initialize new milestone |
| complete-milestone.md | Archive and tag |
| discovery-phase.md | Determine research depth |
| discuss-milestone.md | Gather milestone context |
| discuss-phase.md | Gather phase context |
| list-phase-assumptions.md | Preview Claude's approach |
| map-codebase.md | Analyze existing code |
| research-phase.md | Deep ecosystem research |
| resume-project.md | Continue from handoff |
| transition.md | Phase-to-phase logic |

### Key Workflow: plan-phase.md

```
Input: Phase number
    ↓
[Mandatory Discovery - Level 0-3 based on risk]
    ↓
[Read project history from prior SUMMARY frontmatter]
    ↓
[Break phase into 2-3 atomic tasks]
    ↓
[Estimate scope, may split into multiple plans]
    ↓
Output: XX-YY-PLAN.md (executable prompt)
```

### Key Workflow: execute-phase.md

```
Input: PLAN.md path
    ↓
[Load project state]
    ↓
[Parse plan into checkpoint-delimited segments]
    ↓
[Route to execution strategy]
    │
    ├─ Strategy A: No checkpoints → Single subagent
    ├─ Strategy B: Verify checkpoints → Segmented
    └─ Strategy C: Decision checkpoints → Main context
    ↓
[Atomic commit after each task]
    ↓
[Create SUMMARY.md, update STATE.md]
    ↓
Output: Committed code + SUMMARY.md
```

### Discovery Depth Levels

| Level | When | Duration |
|-------|------|----------|
| 0 (Skip) | Pure internal work, established patterns | - |
| 1 (Quick) | Single known library, syntax check | ~5 min |
| 2 (Standard) | New library/API, choose between options | 15-30 min |
| 3 (Deep) | Architectural decision, niche domain | 1+ hours |

**Risk Indicators:**
- New external library (not in package.json) → Level 2
- New external API → Level 2
- "Choose/select/evaluate" in description → Level 2+
- Niche domains (3D, games, audio, ML) → Level 3

---

## Reference Documents

Reference files provide reusable guidance loaded via `@` syntax.

### Reference Index

| File | Content |
|------|---------|
| principles.md | Core philosophy, scope control, atomic commits |
| questioning.md | Question framework for context gathering |
| plan-format.md | PLAN.md structure, task anatomy |
| checkpoints.md | Checkpoint types: verify (90%), decision (9%), action (1%) |
| scope-estimation.md | Task sizing (15-60 min), when to split |
| tdd.md | TDD structure: RED→GREEN→REFACTOR |
| git-integration.md | Per-task commits, format, rationale |
| continuation-format.md | Handoff context presentation |
| research-pitfalls.md | Common research mistakes |

### Checkpoint Distribution

```
human-verify:  90% - Pause to confirm output
decision:       9% - User chooses path forward
human-action:   1% - User must do something external
```

---

## Context Engineering

Context engineering is GSD's core insight: **quality comes from rich, relevant information, not process**.

### The @path/to/file.md System

Commands include context references:

```markdown
<execution_context>
@.planning/PROJECT.md           # Project vision
@.planning/ROADMAP.md           # Phase structure
@.planning/phases/02-auth/DISCOVERY.md  # Phase research
@src/lib/db.ts                  # Relevant source
</execution_context>
```

When Claude executes:
1. Files are read via the `@` reference
2. Content injected into prompt
3. Full file content available (within token limits)

### Smart Context Selection

The plan-phase workflow builds dependency graphs from SUMMARY frontmatter:

```yaml
# SUMMARY.md frontmatter
phase: 02-auth
subsystem: authentication
requires: [01-foundation]
provides: [user-model, jwt-auth]
affects: [03-dashboard, 05-checkout]
tags: [auth, security]
```

Auto-selection rules:
- Current phase in prior "affects" field → include
- Shared subsystem → include
- In "requires" chain → include

### Context Window Management

GSD respects Claude's quality degradation curve:

```
0-30%  context usage → Peak quality
30-50% context usage → Good quality
50-70% context usage → Degrading
70%+   context usage → Poor quality
```

**Solution:** Keep plans to 2-3 tasks, stay under 50% usage.

---

## Configuration System

### config.json Structure

```json
{
  "mode": "interactive|yolo",
  "depth": "quick|standard|comprehensive",
  "gates": {
    "confirm_project": true,
    "confirm_phases": true,
    "confirm_roadmap": true,
    "confirm_breakdown": true,
    "confirm_plan": true,
    "execute_next_plan": true,
    "issues_review": true,
    "confirm_transition": true
  },
  "safety": {
    "always_confirm_destructive": true,
    "always_confirm_external_services": true
  }
}
```

### Mode Options

| Mode | Behavior |
|------|----------|
| interactive | Confirms each major decision, pauses at checkpoints |
| yolo | Auto-approves most decisions, only critical checkpoints |

### Depth Options

| Depth | Phases | Plans/Phase |
|-------|--------|-------------|
| quick | 3-5 | 1-3 |
| standard | 5-8 | 3-5 |
| comprehensive | 8-12 | 5-10 |

---

## Git Integration Strategy

### Per-Task Atomic Commits

Git history is the primary context source for future sessions.

```
Each task    → 1 commit immediately after completion
Each plan    → 1 metadata commit after all tasks done
```

### Commit Format

```
{type}({phase}-{plan}): {task-name}
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `test` - Tests
- `refactor` - Code restructure
- `perf` - Performance
- `chore` - Maintenance
- `docs` - Documentation

### Example Git History

```
abc123f docs(08-02): complete user registration plan
def456g feat(08-02): add email confirmation flow
hij789k feat(08-02): implement password hashing
lmn012o feat(08-02): create registration endpoint
```

---

## Subagent Architecture

### Execution Strategy Selection

```
┌─────────────────────────────────────────────────────────┐
│              PLAN.md Analysis                            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  NO checkpoints?                                         │
│      ↓ YES                                               │
│  Strategy A: Spawn single subagent                       │
│      - Fresh 200k context                                │
│      - Executes all tasks                                │
│      - Commits each task                                 │
│      - Creates SUMMARY.md                                │
│      - Returns to main context                           │
│                                                          │
│  HAS verify-only checkpoints?                            │
│      ↓ YES                                               │
│  Strategy B: Segmented execution                         │
│      - Subagent for autonomous segments                  │
│      - Main context for verify checkpoints               │
│                                                          │
│  HAS decision checkpoints?                               │
│      ↓ YES                                               │
│  Strategy C: Main context only                           │
│      - Decisions affect subsequent flow                  │
│      - Cannot parallelize                                │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Context Efficiency

| Strategy | Main Context Usage | Subagent Context |
|----------|-------------------|------------------|
| A | ~5% (orchestration) | Full plan + refs |
| B | ~20% (checkpoints) | Segments only |
| C | Full execution | - |

---

## Key Design Patterns

### Pattern 1: Plans Are Prompts

PLAN.md IS the executable prompt. No transformation.

```xml
<task type="auto">
  <name>Task 1: Create login endpoint</name>
  <files>src/api/auth/login.ts</files>
  <action>POST endpoint accepting {email, password}.
          Use bcrypt for comparison.
          Return JWT in httpOnly cookie.</action>
  <verify>curl -X POST localhost:3000/api/auth/login returns 200</verify>
  <done>Valid credentials → 200 + cookie, invalid → 401</done>
</task>
```

### Pattern 2: Decimal Phase Numbering

```
Integer phases: Planned work (1, 2, 3, 4)
Decimal phases: Urgent insertions (2.1, 2.2, 7.1)

Filesystem sort: 1 < 2 < 2.1 < 2.2 < 3
```

Validation:
- Integer X must exist and be complete
- X+1 must exist
- Decimal X.Y must not exist
- Y >= 1

### Pattern 3: TDD Gets Dedicated Plans

TDD requires 2-3 execution cycles:

```
RED     → write failing test (commit: test)
GREEN   → implement to pass (commit: feat)
REFACTOR→ cleanup (commit: refactor)
```

Uses 40-50% context for single feature → dedicated plan per feature.

### Pattern 4: Living Documents

PROJECT.md, STATE.md, ROADMAP.md evolve:

```
- Requirements validated/invalidated
- Decisions logged
- Position tracked
- Blockers recorded
```

Each includes "Last updated" footer.

### Pattern 5: Deviation Rules

During execution:

| Situation | Action |
|-----------|--------|
| Bug found | Auto-fix, document in summary |
| Critical fix needed | Auto-add, document |
| Blocker encountered | Auto-fix, document |
| Architectural change | Stop, ask user |
| Enhancement idea | Log to ISSUES.md, continue |

---

## Complete File Hierarchy

```
get-shit-done/
├── bin/
│   └── install.js                          # NPX installer
│
├── commands/gsd/                           # Slash commands
│   ├── new-project.md
│   ├── create-roadmap.md
│   ├── map-codebase.md
│   ├── plan-phase.md
│   ├── execute-plan.md
│   ├── progress.md
│   ├── discuss-phase.md
│   ├── research-phase.md
│   ├── list-phase-assumptions.md
│   ├── add-phase.md
│   ├── insert-phase.md
│   ├── new-milestone.md
│   ├── complete-milestone.md
│   ├── discuss-milestone.md
│   ├── pause-work.md
│   ├── resume-work.md
│   ├── consider-issues.md
│   └── help.md
│
├── get-shit-done/
│   ├── references/                         # Guidance documents
│   │   ├── principles.md
│   │   ├── questioning.md
│   │   ├── plan-format.md
│   │   ├── checkpoints.md
│   │   ├── scope-estimation.md
│   │   ├── tdd.md
│   │   ├── git-integration.md
│   │   ├── continuation-format.md
│   │   └── research-pitfalls.md
│   │
│   ├── templates/                          # File templates
│   │   ├── project.md
│   │   ├── state.md
│   │   ├── roadmap.md
│   │   ├── phase-prompt.md
│   │   ├── summary.md
│   │   ├── context.md
│   │   ├── discovery.md
│   │   ├── research.md
│   │   ├── issues.md
│   │   ├── milestone.md
│   │   ├── continue-here.md
│   │   ├── milestone-context.md
│   │   ├── milestone-archive.md
│   │   ├── config.json
│   │   └── codebase/
│   │       ├── stack.md
│   │       ├── architecture.md
│   │       ├── structure.md
│   │       ├── conventions.md
│   │       ├── testing.md
│   │       ├── integrations.md
│   │       └── concerns.md
│   │
│   └── workflows/                          # Multi-step procedures
│       ├── plan-phase.md
│       ├── execute-phase.md
│       ├── create-roadmap.md
│       ├── create-milestone.md
│       ├── complete-milestone.md
│       ├── discovery-phase.md
│       ├── discuss-milestone.md
│       ├── discuss-phase.md
│       ├── list-phase-assumptions.md
│       ├── map-codebase.md
│       ├── research-phase.md
│       ├── resume-project.md
│       └── transition.md
│
├── docs/
│   ├── SECURITY.md
│   └── RESEARCH-arch.md                    # This document
│
├── package.json
├── CLAUDE.md
├── README.md
├── LICENSE
└── .gitignore
```

---

## Project Lifecycle Example

### Complete Flow: E-commerce App

```
1. INITIALIZATION
   $ /gsd:new-project
   → Creates: .planning/PROJECT.md, .planning/config.json
   → Commits: "docs: initialize ecommerce-app"

2. ROADMAP CREATION
   $ /gsd:create-roadmap
   → Creates: .planning/ROADMAP.md (5-8 phases)
   → Creates: .planning/STATE.md
   → Creates: .planning/phases/{01,02,03,...}/
   → Commits: "docs: initialize ecommerce-app (5 phases)"

3. PHASE 1: FOUNDATION
   $ /gsd:discuss-phase 1
   → Creates: .planning/phases/01-foundation/01-CONTEXT.md

   $ /gsd:plan-phase 1
   → Creates: .planning/phases/01-foundation/01-01-PLAN.md

   $ /gsd:execute-plan .planning/phases/01-foundation/01-01-PLAN.md
   → Commits: feat(01-01): create Next.js project
   → Commits: feat(01-01): configure Prisma
   → Commits: feat(01-01): set up Tailwind
   → Creates: .planning/phases/01-foundation/01-01-SUMMARY.md
   → Commits: "docs(01-01): complete foundation plan"

4. URGENT FIX (Mid-project)
   $ /gsd:insert-phase 2 "Fix CORS bug"
   → Creates: Phase 2.1

   $ /gsd:plan-phase 2.1
   $ /gsd:execute-plan ...
   → Commits: fix(02.1-01): fix CORS headers

5. MILESTONE COMPLETION
   $ /gsd:complete-milestone 1.0.0
   → Archives to milestones/
   → Creates: git tag v1.0.0
   → Commits: "docs: ship v1.0.0"

6. NEXT MILESTONE
   $ /gsd:new-milestone "v1.1 Admin Dashboard"
   → Continues cycle
```

---

## Summary

GSD transforms Claude Code from a code assistant into a structured execution engine through:

1. **Meta-Prompting**: Commands generate precise prompts for Claude
2. **Context Engineering**: Smart `@file` inclusion provides relevant information
3. **Atomic Execution**: Fresh context windows prevent quality degradation
4. **Living Documents**: PROJECT/STATE/ROADMAP evolve with the project
5. **Git Integration**: Per-task commits optimize for AI workflows
6. **Subagent Spawning**: Efficient context usage through delegation

The system is designed for **solo developers** who want Claude Code as their primary implementer, eliminating enterprise ceremony while maximizing implementation quality through careful scope and context management.

---

*Last updated: 2026-01-06*
