---
layout: post
title: "Spec-Kit Extensions: Making spec-driven development your own"
date: 2026-04-08
categories: [AI, Development, DevOps]
tags: [ai-assisted-development, spec-kit, github, copilot, extensions, specification-driven-development, ralph-loop]
author: hidde
description: "Spec-Kit's extension system turns a structured spec-driven workflow into something you can customize end-to-end. Here's how extensions work, which ones are worth looking at, and a hands-on walkthrough of the Ralph Loop extension for autonomous implementation."
image: /images/spec-kit/speckit-extensions.png
featured: false
toc: true
---

The core Spec-Kit workflow (constitution, specify, plan, tasks, implement) gets you far. But every team has different needs. Some want Jira integration. Others want post-implementation code review. A few want an autonomous loop that implements every task while they grab coffee.

Spec-Kit ships with a modular extension system that lets you bolt on new commands, templates, and workflows without touching the core. The community has already built over 40 extensions, and the catalog keeps growing. In this post, I'll walk through how extensions work, highlight a handful worth knowing, and dig into the **Ralph Loop** extension as an example.

---

## What are Spec-Kit extensions?

Extensions add new capabilities to Spec-Kit. Presets customize existing behavior. Quick comparison:

| Mechanism | Purpose | Example |
|-----------|---------|---------|
| **Extension** | Add new commands or workflows | Jira sync, code review, autonomous loops |
| **Preset** | Customize specs, plans, and task formats | Pirate speak, compliance templates, TOC navigation |

Extensions live in the `extensions/` directory and follow a standard structure: a manifest file (`extension.yml`), command templates, optional scripts, and configuration. You install them with the CLI:

```bash
# Search what's available
specify extension search

# Install an extension
specify extension add <extension-name>

# List installed extensions
specify extension list
```

That's it. The commands from the extension become available as slash commands in your AI agent session, just like the built-in ones.

---

## Extensions worth knowing about

The [community catalog](https://speckit-community.github.io/extensions/) has extensions across five categories: **docs**, **code**, **process**, **integration**, and **visibility**. Here are a few worth highlighting.

### Docs extensions

| Extension | What it does |
|-----------|-------------|
| **Iterate** | Refine specs mid-implementation and go straight back to building |
| **Reconcile** | Surgically updates feature artifacts when implementation drifts from spec |
| **Spec Critique** | Dual-lens critical review from product strategy and engineering risk perspectives |

### Code extensions

| Extension | What it does |
|-----------|-------------|
| **Review** | Post-implementation code review with specialized agents for quality, tests, error handling, and simplification |
| **Staff Review** | Staff-engineer-level review validating implementation against spec, checking security and performance |
| **Fix Findings** | Automated analyze-fix-reanalyze loop that resolves spec findings until clean |
| **Cleanup** | Post-implementation quality gate. Fixes small issues, creates tasks for medium ones |
| **Verify** | Validates implemented code against specification artifacts |

### Process extensions

| Extension | What it does |
|-----------|-------------|
| **Fleet Orchestrator** | Orchestrates a full feature lifecycle with human-in-the-loop gates across all SpecKit phases |
| **MAQA** | Coordinator > feature > QA agent workflow with parallel worktree-based implementation |
| **Conduct** | Delegates spec-kit phases to sub-agents to reduce context pollution |
| **Product Forge** | Full product lifecycle: research > product spec > SpecKit > implement > verify > test |

### Integration extensions

| Extension | What it does |
|-----------|-------------|
| **Jira Integration** | Creates Jira Epics, Stories, and Issues from spec-kit specs and task breakdowns |
| **Azure DevOps** | Syncs user stories and tasks to Azure DevOps work items |
| **Confluence** | Creates a Confluence doc summarizing specs and planning files |

### Visibility extensions

| Extension | What it does |
|-----------|-------------|
| **Project Health Check** | Diagnoses a Spec Kit project and reports health issues across structure, agents, features, scripts, extensions, and git |
| **Project Status** | Shows current SDD workflow progress: active feature, artifact status, task completion, and extensions summary |

Each extension declares whether it's **Read-only** (reports without modifying files) or **Read+Write** (modifies files, creates artifacts). Check the label before you install something that writes to your repo.

That's a lot of options. Rather than staying at catalog level, let's pick one extension and look at how it actually works end-to-end.

---

## The Ralph Loop: autonomous implementation in practice

Most extensions add a review step or a sync capability. The [Ralph Loop](https://github.com/Rubiss/spec-kit-ralph) goes further: it takes your `tasks.md` and implements everything autonomously, task by task, in a loop.

### How it works

Ralph spawns a fresh AI agent process for each iteration. The agent reads `tasks.md`, picks the first incomplete work unit, implements it, marks the tasks as done, commits the result, and hands control back to the orchestrator. The orchestrator checks termination conditions and loops.

```
┌─────────────────────────────────────────┐
│           ralph-loop starts             │
│  validate prerequisites, load config    │
└──────────────────┬──────────────────────┘
                   ▼
          ┌────────────────┐
          │ Any tasks left?│──No──▶ exit 0 (COMPLETED)
          └───────┬────────┘
                  │ Yes
                  ▼
  ┌───────────────────────────────┐
  │  Spawn fresh agent process    │
  │  copilot --agent speckit.ralph│
  └──────────────┬────────────────┘
                 ▼
  ┌───────────────────────────────┐
  │  Agent reads tasks.md +       │
  │  progress.md, implements      │
  │  ONE work unit, commits       │
  └──────────────┬────────────────┘
                 ▼
       ┌──────────────────┐
       │ Check termination│
       │   conditions     │
       └────────┬─────────┘
                ▼
        back to "Any tasks left?"
```

Each iteration is self-contained. If you interrupt with Ctrl+C, you can resume later. Ralph reads the checkbox state in `tasks.md` and skips completed tasks. The `progress.md` log gives the agent context from prior iterations.

### What it looks like in the terminal

When you kick off a run, the orchestrator prints a header per iteration and streams the agent's output in real-time:

```
============================================================
  Ralph Loop - 001-my-feature
  Iteration 1 of 10
============================================================

[09:12:03] * Running   - Starting iteration

--- Copilot Agent Output ---
Reading tasks.md... found 12 tasks across 3 user stories.
Working on US-001: Initialize project structure
  [x] T001: Create directory layout
  [x] T002: Add configuration files
  [x] T003: Set up dependency management
Committing: feat(001-my-feature): US-001 Initialize project structure
--- End Agent Output ---

[09:14:27] * Success   - Iteration completed
9 task(s) remaining

============================================================
  Ralph Loop - 001-my-feature
  Iteration 2 of 10
============================================================

[09:14:28] * Running   - Starting iteration

--- Copilot Agent Output ---
Reading tasks.md... found 9 tasks across 2 user stories.
Working on US-002: Implement core API endpoints
  [x] T004: Create REST controller
  [x] T005: Add request validation
  [x] T006: Write unit tests
  [x] T007: Add integration tests
Committing: feat(001-my-feature): US-002 Implement core API endpoints
--- End Agent Output ---

[09:18:41] * Success   - Iteration completed
5 task(s) remaining
```

Each iteration gets its own banner, and you can see exactly which tasks are being checked off. When all tasks are done (or the agent hits a termination condition), the orchestrator prints a summary and exits.

### What progress.md looks like

After each iteration, the agent appends an entry to `progress.md` in the spec directory. This file serves two purposes: it gives you a log of what happened, and it gives the next iteration context about what was already done.

```markdown
---
## Iteration 1 - 2026-04-08 09:14:27
**User Story**: US-001 Initialize project structure
**Tasks Completed**:
- [x] T001: Create directory layout
- [x] T002: Add configuration files
- [x] T003: Set up dependency management
**Tasks Remaining in Story**: None - story complete
**Commit**: a1b2c3d
**Files Changed**:
- src/config/app.yml
- src/config/database.yml
- package.json
**Learnings**:
- Project uses ESM modules, not CommonJS
- Prettier config already exists in repo root

---
## Iteration 2 - 2026-04-08 09:18:41
**User Story**: US-002 Implement core API endpoints
**Tasks Completed**:
- [x] T004: Create REST controller
- [x] T005: Add request validation
- [x] T006: Write unit tests
- [x] T007: Add integration tests
**Tasks Remaining in Story**: None - story complete
**Commit**: d4e5f6a
**Files Changed**:
- src/controllers/items.controller.ts
- src/validators/items.validator.ts
- tests/unit/items.test.ts
- tests/integration/items.integration.test.ts
**Learnings**:
- Existing tests use vitest, not jest
- Validation uses zod schemas from src/validators/
```

The "Learnings" section is useful. The agent picks up conventions from the codebase during each iteration and writes them down, so the next fresh agent process doesn't repeat the same discovery work.

### Installing Ralph

```bash
# Install from the community catalog
specify extension add ralph

# Or install directly from the repository
specify extension add ralph --from \
  https://github.com/Rubiss/spec-kit-ralph/archive/refs/tags/v1.0.0.zip

# Verify
specify extension list
# ✓ Ralph Loop (v1.0.0)
#   Autonomous implementation loop using AI agent CLI
#   Commands: 2 | Hooks: 1 | Status: Enabled
```

### Running it

Two paths. Inside an agent session:

```
/speckit.ralph.run
```

With options:

```
/speckit.ralph.run --max-iterations 5 --model gpt-5.1
```

Or directly from the terminal for debugging or CI use:

```bash
.specify/extensions/ralph/scripts/bash/ralph-loop.sh \
  --feature-name "001-my-feature" \
  --tasks-path "specs/001-my-feature/tasks.md" \
  --spec-dir "specs/001-my-feature" \
  --max-iterations 10 \
  --model "claude-sonnet-4.6"
```

### Configuration

Ralph uses a layered config system. Edit `.specify/extensions/ralph/ralph-config.yml` for project defaults:

```yaml
# AI model for agent iterations
model: "claude-sonnet-4.6"

# Maximum loop iterations before stopping
max_iterations: 10

# Path or name of the agent CLI binary
agent_cli: "copilot"
```

Settings resolve from lowest to highest priority:

| Priority | Source |
|----------|--------|
| 1 (lowest) | Extension defaults in `extension.yml` |
| 2 | Project config `.specify/extensions/ralph/ralph-config.yml` |
| 3 | Local overrides `.specify/extensions/ralph/ralph-config.local.yml` (gitignored) |
| 4 | Environment variables (`SPECKIT_RALPH_MODEL`) |
| 5 (highest) | CLI parameters (`--model`, `--max-iterations`) |

Local overrides and environment variables mean each developer on the team can use their preferred model without committing changes.

### Termination conditions

Ralph stops for one of five reasons:

| Condition | Exit code | Meaning |
|-----------|-----------|---------|
| All tasks marked `[x]` | 0 | Done |
| Agent outputs `<promise>COMPLETE</promise>` | 0 | Agent confirmed completion |
| Max iterations reached | 1 | Safety limit, increase if needed |
| 3 consecutive failures | 1 | Circuit breaker, agent is stuck |
| Ctrl+C | 130 | User interrupted |

The circuit breaker at three consecutive failures is a practical safety net. If the agent can't make progress, it stops rather than burning tokens in an infinite loop.

### When Ralph makes sense

Ralph works best when:

- Your `tasks.md` is well-structured with clear, independent work units
- Tasks are granular enough for one agent iteration each
- You've already validated your spec and plan
- You're comfortable reviewing commits after the fact rather than watching each one

It's not a replacement for understanding what's being built. But for teams that have already invested in thorough specs and plans, Ralph turns those artifacts into implemented code without manual shepherding.

---

## Building your own extension

If the community catalog doesn't cover your use case, you can build and publish your own. Here's the structure:

```
my-extension/
├── extension.yml          # Manifest: name, version, commands, hooks
├── commands/
│   └── my-command.md      # Command template (becomes /speckit.ext.my-command)
├── scripts/
│   └── bash/
│       └── my-script.sh   # Optional automation scripts
├── README.md
└── LICENSE
```

The `extension.yml` manifest declares your commands, hooks, and metadata. Once published, others can install it with `specify extension add`. The [Extension Publishing Guide](https://github.com/github/spec-kit/blob/main/extensions/EXTENSION-PUBLISHING-GUIDE.md) covers the full process.

---

## Where to start

- Browse the [community catalog](https://speckit-community.github.io/extensions/) to see what's out there
- Install one or two extensions that fill gaps in your current workflow
- If nothing fits, the extension format is simple enough to build your own in an afternoon

The core Spec-Kit workflow gives you structure. Extensions give you the specific tooling your team actually needs, from Jira sync to autonomous implementation. Ralph Loop shows what happens when you take the spec-driven approach to its logical end: you write the what, the loop handles the how.
