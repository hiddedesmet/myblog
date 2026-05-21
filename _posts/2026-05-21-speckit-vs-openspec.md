---
layout: post
title: "Spec-Kit vs OpenSpec: two takes on spec-driven development"
date: 2026-05-21
categories: [AI, Development, DevOps]
tags: [ai-assisted-development, spec-kit, openspec, github, copilot, claude, spec-driven-development, comparison]
author: hidde
description: "Spec-Kit and OpenSpec are two prominent spec-driven workflows for AI coding agents. They look similar on the surface and diverge sharply once you actually use them. Here's how they compare on workflow, philosophy, brownfield support, and customization."
image: /images/speckit-vs-openspec.png
featured: true
toc: true
---

I spoke at Techorama last week about spec-driven development. [Geert van der Cruijsen](https://www.linkedin.com/in/geertvandercruijsen/) was speaking on the same topic in the same timeslot, so I missed his session. That annoyed me enough that I went home and dug into OpenSpec, the tool he'd been talking up, to see what I'd been missing. This post is the result.

Two projects keep showing up whenever spec-driven development comes up: GitHub's [Spec-Kit](https://github.com/github/spec-kit) and Fission AI's [OpenSpec](https://github.com/Fission-AI/OpenSpec). Both are widely adopted, both promise the same outcome: stop vibe-coding, agree on what to build before the agent writes code. They get there in very different ways.

I've been running Spec-Kit on real projects for a while. OpenSpec is newer to me. This post is what I figured out comparing them side by side.

<div class="tip" markdown="1">
**Short version:** use Spec-Kit when you want a guided workflow, strong guardrails, and a big extension catalog. Use OpenSpec when you are working in an existing codebase and want changes to be written as deltas against current behavior.
</div>

---

## TL;DR

If you want a quick verdict:

| | Spec-Kit | OpenSpec |
|---|---|---|
| **Maintainer** | GitHub | Fission AI |
| **Language / runtime** | Python (uv / pipx) | TypeScript (npm) |
| **Workflow shape** | Linear phases with commands or skills | Fluid actions on a DAG of artifacts |
| **Greenfield fit** | Excellent | Good |
| **Brownfield fit** | Workable | Excellent (delta-spec first-class) |
| **Customization** | Extensions + presets + overrides | Schemas + project config |
| **Best for** | Structured teams, 0→1 work, enterprise constraints | Iterative work, brownfield modifications, parallel changes |

Both are MIT-licensed, both support a long list of AI coding assistants, and both are evolving fast. The real choice comes down to whether you want phases with guardrails or actions on a graph.

---

## What each one actually is

### Spec-Kit

Spec-Kit ships a CLI (`specify`) that installs commands, skills, and project guidance for your AI coding agent of choice. The README puts it bluntly:

> Spec-Driven Development flips the script on traditional software development. For decades, code has been king. Specifications were just scaffolding we built and discarded once the "real work" of coding began.

The flow is linear and named:

```text
/speckit.constitution → /speckit.specify → /speckit.plan → /speckit.tasks → /speckit.implement
```

Optional commands fill in the gaps: `/speckit.clarify` for under-specified areas, `/speckit.analyze` for cross-artifact consistency, `/speckit.checklist` for "unit tests for English". The feature artifacts live under `specs/<feature>/`, while `.specify/` holds the templates, scripts, integration state, and customization layers that drive the workflow.

The philosophy is explicit: "Intent-driven development where specifications define the *what* before the *how*", and "Multi-step refinement rather than one-shot code generation from prompts."

### OpenSpec

OpenSpec ships an npm package (`@fission-ai/openspec`) that installs agent skills and, where supported, command files into your project. Its tagline gives away the bias:

```text
→ fluid not rigid
→ iterative not waterfall
→ easy not complex
→ built for brownfield not just greenfield
```

The current default is the **OPSX workflow** with five commands in the `core` profile: `/opsx:propose`, `/opsx:explore`, `/opsx:apply`, `/opsx:sync`, `/opsx:archive`. An expanded profile adds `/opsx:new`, `/opsx:continue`, `/opsx:ff`, `/opsx:verify`, `/opsx:bulk-archive`, and `/opsx:onboard`.

The structural unit is a **change folder** under `openspec/changes/`, each containing a proposal, optional design, tasks, and **delta specs** that describe what changes relative to the current state. When you archive, deltas merge into `openspec/specs/`, which is your source of truth.

---

## The structural difference that matters

Most comparisons skip this part. Spec-Kit and OpenSpec organize artifacts in fundamentally different ways.

**Spec-Kit** treats each feature as a self-contained spec directory. You run `/speckit.specify` for what you want, `/speckit.plan` for how, `/speckit.tasks` for the breakdown, `/speckit.implement` to execute. The constitution sets project-wide principles, but the core workflow does not maintain a domain-by-domain source of truth in the same way OpenSpec does. Each new feature starts from its own numbered spec folder.

**OpenSpec** maintains a permanent `specs/` directory as the system's source of truth, and every change is a **delta** against it:

```text
openspec/changes/add-2fa/specs/auth/spec.md

## ADDED Requirements
### Requirement: Two-Factor Authentication
The system MUST support TOTP-based two-factor authentication.

## MODIFIED Requirements
### Requirement: Session Expiration
The system MUST expire sessions after 15 minutes of inactivity.
(Previously: 30 minutes)

## REMOVED Requirements
### Requirement: Remember Me
(Deprecated in favor of 2FA.)
```

On archive, those deltas merge into `openspec/specs/auth/spec.md`, and the next change builds on the updated baseline. It's git-like: specs are the main branch, changes are feature branches with explicit diffs.

For brownfield work, this matters a lot. You're not restating an entire auth system every time you touch it. You're describing what's being added, modified, or removed. For greenfield, it matters less.

Worth flagging: Spec-Kit doesn't do this out of the box, but the community [spec-kit-sync extension](https://speckit-community.github.io/extensions/sync) closes part of the gap. It detects drift between specs and code (`/speckit.sync.analyze`), proposes resolutions, and can backfill specs from unspecced features. That's a different shape from OpenSpec's deltas. Sync reconciles after the fact, deltas are the authoring model. But if you're on Spec-Kit and staring at a drifted codebase, it's one of the relevant brownfield options to look at.

---

## Workflow philosophy: phases vs actions

Spec-Kit presents a forward path. The named order (constitution, specify, plan, tasks, implement) is the path. You *can* go back and edit, but the core workflow doesn't model iteration as a first-class concept in the same way OpenSpec does. The optional `/speckit.analyze` command exists specifically to catch drift between artifacts before you implement.

OpenSpec rejects phases entirely. From the OPSX docs:

> Actions, not phases. Create, implement, update, archive. Do any of them anytime. Dependencies are enablers: they show what's possible, not what's required next.

Under the hood OpenSpec maintains a DAG of artifacts (`proposal → specs → design → tasks → implement`). The CLI knows which artifacts are `ready`, `blocked`, or `done` based on what exists on disk, and the generated workflow instructions can query that state when they need richer context. If you realize mid-implementation that the design is wrong, you edit `design.md` and continue. The agent picks up where it left off.

Which is better depends on how you actually work. If your team needs guardrails ("no implementing until tasks are reviewed"), Spec-Kit's structure helps. If you want to bounce between proposal, spec, and code as you learn, OpenSpec gets out of the way.

---

## Customization: extensions vs schemas

Both projects let you change how they behave. They picked opposite mechanisms.

**Spec-Kit** uses a layered system: core templates ship with the CLI, **presets** override existing templates (e.g. compliance-format specs, different terminology, security gates), and **extensions** add entirely new commands (Jira sync, V-Model traceability, Ralph loops for autonomous implementation). Resolution order is project-local overrides → presets → extensions → core. The community catalog now lists dozens of extensions.

I covered the extension model in detail in [Spec-Kit Extensions](/speckit-extensions.html). It's powerful, and it's the reason Spec-Kit scales to enterprise constraints.

**OpenSpec** moves customization down a level: you edit the **schema** itself. The default is `spec-driven` (proposal → specs → design → tasks), but you can fork it or write your own:

```yaml
# openspec/schemas/research-first/schema.yaml
name: research-first
artifacts:
  - id: research
    generates: research.md
    requires: []
  - id: proposal
    generates: proposal.md
    requires: [research]
  - id: tasks
    generates: tasks.md
    requires: [proposal]
```

Add a `research` artifact before `proposal`, drop `design` if you don't want it, define your own dependency graph. A project-level `openspec/config.yaml` injects context and per-artifact rules into every prompt.

The trade-off: Spec-Kit gives you a marketplace of prebuilt extensions. OpenSpec gives you a smaller, more direct lever, but you have to author the schema yourself.

---

## Tooling and install footprint

Practical stuff that matters when you onboard a team:

| | Spec-Kit | OpenSpec |
|---|---|---|
| **Runtime** | Python 3.11+ via `uv` or `pipx` | Node.js 20.19+ via `npm`, `pnpm`, `yarn`, `bun`, or `nix` |
| **Install** | `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z` | `npm install -g @fission-ai/openspec@latest` |
| **Project init** | `specify init my-project --integration copilot` | `openspec init` |
| **Agent integrations** | 30+ (Copilot, Claude Code, Cursor, Codex, Windsurf, Gemini, Junie, etc.) | 25+ (similar coverage) |
| **State on disk** | `.specify/` plus feature folders under `specs/` | `openspec/specs/`, `openspec/changes/`, `openspec/schemas/` |

Spec-Kit's Python footprint can feel heavier if your team isn't already using uv. OpenSpec's Node install is one line and a path most JS/TS teams already have.

Both write Markdown to your repo. Both can refresh generated agent instructions after upgrades (`specify integration upgrade` for Spec-Kit, `openspec update` for OpenSpec). Both work cross-platform.

---

## A worked example: adding 2FA to an existing auth system

The same task in both tools.

**Spec-Kit:**

```text
/speckit.specify Add TOTP-based two-factor authentication. Users
  enroll via QR code, verify with a code during enrollment, and are
  prompted for OTP at login. 30-minute session timeout becomes 15
  minutes when 2FA is active.

/speckit.clarify         # surfaces gaps: recovery codes? SMS fallback?
/speckit.plan            # tech stack, libraries, data model
/speckit.tasks           # breakdown
/speckit.analyze         # cross-check consistency
/speckit.implement       # execute
```

You end up with a fresh numbered feature folder, for example `specs/003-add-2fa/spec.md`, describing the feature in isolation. The original auth spec isn't modified. It sits alongside.

**OpenSpec:**

```text
/opsx:propose add-2fa

# Creates openspec/changes/add-2fa/
#   ├── proposal.md
#   ├── design.md
#   ├── tasks.md
#   └── specs/auth/spec.md   ← delta, not full spec
```

The delta spec uses `## ADDED Requirements`, `## MODIFIED Requirements`, and `## REMOVED Requirements` to describe exactly what changes against the existing `openspec/specs/auth/spec.md`. After `/opsx:apply` and `/opsx:archive`, the deltas merge in and the auth spec now reflects 2FA as part of its definition.

If you later want to remove 2FA, the change folder still exists in `changes/archive/`, so you can read the proposal that explained *why* it was added in the first place.

---

## Where each one wins

**Pick Spec-Kit when:**

- You're starting greenfield projects from a clean slate
- You want named phases and explicit gates between planning and implementation
- You need an extension ecosystem (Jira, code review, autonomous loops, V-Model)
- Your team benefits from a constitution that governs all subsequent work
- You're in a Python-friendly environment

**Pick OpenSpec when:**

- You work mostly in existing codebases and want delta specs
- You want to bounce between artifacts as you learn, not march through phases
- Multiple changes will be in flight in parallel
- Your team is TypeScript/Node-native and an npm install is the friction-free path
- You want fewer ceremony layers: specs, changes, archive

**Pick neither when:**

- The work is throwaway. A prototype you'll delete tomorrow doesn't need a spec. Both projects say so.

---

## What surprised me

A few things I didn't expect after running both:

1. **OpenSpec's delta model changed how I review AI work.** Reading a delta is faster than diffing a regenerated full spec. For brownfield work this is a bigger deal than I thought.
2. **Spec-Kit's `/speckit.analyze` catches more drift than I expected.** Running it before `/speckit.implement` has saved me from at least three "the spec says X, the plan says Y" rewrites.
3. **Both tools are learning from similar pressure.** OpenSpec has community schemas that work a bit like Spec-Kit's extension catalog. Spec-Kit's extension ecosystem also fills brownfield and iteration gaps that the core workflow does not cover directly.
4. **Model choice matters more than tool choice.** OpenSpec's docs recommend high-reasoning models for both planning and implementation. That tracks with my experience on Spec-Kit too. Frontier reasoning models usually produce better specs and plans, but they are expensive. I would not burn them on every task. Use them for ambiguous specs, architecture decisions, and final consistency checks. Use cheaper models for mechanical implementation once the plan is clear.

---

## Can you use both?

Technically yes. They write to different directories (`.specify/` vs `openspec/`) and expose different command or skill names (`/speckit.*` vs `/opsx:*`). I wouldn't recommend it in practice. Your agent's context window fills up fast with two competing sets of instructions, and you'll end up choosing one per project anyway.

If you want to experiment, run Spec-Kit on a new greenfield project and OpenSpec on an existing brownfield codebase. The difference in friction will tell you which one fits your work.

---

## Closing thought

Spec-driven development isn't really about the tool. It's about treating the spec as the artifact that survives, and the code as the thing that gets regenerated. Both Spec-Kit and OpenSpec take that idea seriously.

OpenSpec earns its keep on brownfield work because delta specs match how change actually happens in mature codebases. You don't rewrite the auth system, you modify it. Spec-Kit earns its keep on new projects because the constitution-first model and the extension ecosystem give you a runway that's hard to match.

Pick the one that matches the work in front of you. The wrong choice isn't catastrophic. Both produce readable Markdown you can carry between tools. The right choice just means less friction.
