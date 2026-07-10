---
layout: post
title: "The hidden costs of spec-driven development: when structure helps and when it slows you down"
date: 2026-07-10 09:00:00 +0000
tags: [spec-driven-development, ai-assisted-development, spec-kit, engineering-metrics, productivity, governance]
author: hidde
description: "Spec-driven development fixes the 70% problem, but it also adds planning tax, review bottlenecks, and process overhead. Here is how to know when full specs are worth it and when to use a lighter path."
image: /images/hiddencostsdd.png
featured: true
toc: true
---

In one PR, you fix a validation bug in 20 minutes.
In another PR, you spend half a day writing spec, plan, and tasks before touching code.

Both are "spec-driven". Only one is the right trade-off.

I ran a GitHub Copilot training at a bank this week. Regulated environment, heavy change control, the kind of place where "where's the audit trail" is a normal question in a design review. Partway through, someone asked the question I now get almost every time: **when does spec-driven development actually make sense, and when is it overkill?**

Good question, and not a new one for me, but it landed differently in a room full of people who already live under process. Their instinct wasn't "do we need more structure", it was "we already have too much ceremony, will this add another layer of it".

I still believe in spec-driven development. It solves real problems in AI-assisted coding: scope drift, hidden assumptions, and brittle output that works today but hurts you next month.

But the uncomfortable truth is this: structure is not free.

If you apply full process to every task, you can replace code debt with delivery debt. In a bank, that trade-off is even sharper: the wrong process tax doesn't just slow down a feature team, it slows down every team that has to route work through the same reviewers and the same change board.

This post is about where that cost comes from, how to spot it early, and how to keep the benefits of spec-driven workflows without turning your team into a paperwork factory. It's also my longer answer to that question from the training room.

---

## What you get from this post

By the end, you should be able to:

- identify the hidden cost centers in spec-driven workflows,
- classify work into full, light, or no-spec paths,
- use simple KPIs to decide whether your current process is helping or hurting.

If you have read the earlier series, think of this as the pragmatic sequel: not "why specs matter," but "when specs pay for themselves."

---

## The cost teams forget to track: process overhead

Most engineering dashboards track implementation effort.
Almost none track process overhead.

In a spec-driven flow, overhead usually appears in four places before code lands:

1. Writing and refining the spec
2. Translating spec into a plan
3. Breaking plan into tasks
4. Reviewing artifacts before implementation

For high-risk work, this cost is often worth paying.
For low-risk work, it can be waste.

Quick litmus test:

```text
If "time to first commit" keeps rising,
and escaped defects are flat,
you are likely over-paying process cost.
```

---

## The six hidden costs of spec-driven development

### 1) Planning tax

This is the obvious one.
More structure means more up-front writing and alignment.

**Good when:** requirements are ambiguous or cross-team.
**Bad when:** you are fixing a known issue in one bounded component.

### 2) Context maintenance tax

Specs, plans, tasks, and implementation drift unless someone actively keeps them in sync.

Spec-as-source promises a single truth.
In practice, many teams end up with three "almost current" files.

**Warning signal:** people ask "which file is the source of truth?"

### 3) Review bottleneck tax

Spec-first moves review earlier, which is usually a good quality move.
But it can create queues around the same few senior reviewers.

**Warning signal:** work spends more time waiting for spec approval than it spends being implemented.

### 4) False precision tax

Detailed plans can create fake certainty.
Teams start optimizing for task completion instead of outcome correctness.

**Warning signal:** "We know this is no longer the best path, but it is in tasks.md."

### 5) Tooling friction tax

Workflow quality depends on tooling quality.
If the flow is brittle, context gets lost and process turns into admin work.

**Warning signal:** engineers spend time fixing the process instead of delivering product value.

### 6) Morale tax

When every change requires heavyweight ceremony, people either bypass the process or comply mechanically.

Both outcomes are expensive.

**Warning signal:** "process compliance" rises while confidence in outcomes drops.

---

## Where spec-driven still wins hard

This is not an anti-spec post.
There are cases where full structure is still the strongest option.

| Scenario | Why full specs win |
|----------|--------------------|
| Cross-team feature | Shared contract reduces integration surprises |
| Security-sensitive change | Explicit constraints and review gates |
| Long-lived product area | Durable context for future maintainers |
| AI-generated multi-file work | Better control of scope and architecture |
| Compliance-heavy environments | Clear audit trail and decision history |

The key is not to remove structure.
The key is to apply the right amount of structure.

---

## The question from the training room

Back to that bank. The room was full of engineers who already work inside a change advisory board, mandatory peer review, and audit logging on production systems. Their worry was not "how do we add rigor", it was "we already have rigor, does spec-driven development just bolt a second bureaucracy on top of the first one".

Here is the version of the answer I gave them, and would give again:

- **If your existing change process already demands a written rationale, a reviewed design, and a rollback plan** (which most regulated change boards do), a full spec is not new overhead. It is the same thinking, written in a format an AI agent can actually execute against. You are not adding process, you are formalizing process you already pay for.
- **If the change is internal tooling, a config tweak, or a same-day bug fix that never reaches the change board**, forcing it through `/speckit.specify` → `/speckit.plan` → `/speckit.tasks` adds a second approval queue for work that never needed the first one. That is pure planning tax with no offsetting risk reduction.
- **The deciding question is not "is this a bank" or "is this regulated"**. It's "does this change already require someone else to understand and approve the reasoning before it ships?" If yes, spec-driven development gives that approval something concrete to review instead of a vague ticket description. If no, skip straight to the no-spec lane.

That distinction matters more in regulated environments, not less. When everything is treated as compliance-heavy by default, teams either over-apply full specs to trivial changes (slowing delivery for no risk reduction), or, more often, quietly bypass the process altogether because it feels like duplicate paperwork. Neither is what the change board actually wants. What they want is a spec exactly where the audit trail already required one, and nothing extra where it didn't.

---

## A practical model: full, light, or no-spec

One lane is the problem.
Three lanes usually work better.

| Lane | Use when | Artifacts |
|------|----------|-----------|
| **Full Spec** | High risk, cross-team, security-critical | Constitution checks, full spec, plan, tasks |
| **Light Spec** | Medium risk feature work | Short spec + implementation checklist |
| **No Spec** | Low-risk, reversible changes | Ticket + PR + tests |

If everything is routed through Full Spec, process cost compounds quickly.

---

## Minimum viable light-spec template

For most medium-risk work, this is enough:

1. **Problem**: what is broken or missing
2. **Scope**: what is in and explicitly out
3. **Success criteria**: observable outcomes
4. **Risks**: failure modes and constraints
5. **Validation**: tests/checks before merge

A short, sharp spec often beats a long, vague one.

---

## If Spec-Kit feels too heavy, you have lighter options

Someone in the same training asked a follow-up: "does it have to be Spec-Kit?" No. Spec-Kit is deliberately thorough (constitution, spec, plan, tasks, analyze), which is exactly why it fits the Full Spec lane and feels like overkill everywhere else. Two lighter tools are worth knowing about if that ceremony is the blocker, not the concept of writing things down.

**[OpenSpec](https://openspec.dev/)** bills itself as "a lightweight spec-driven framework." Instead of a full spec/plan/tasks pipeline, each change produces a spec delta: a diff-style document showing exactly what requirements and scenarios changed, such as adding a "Remember me" scenario to a session-expiration spec. It's model-agnostic (no API keys, no MCP server needed), installs with `npm install -g @fission-ai/openspec@latest`, and works with Claude Code, Cursor, Codex, GitHub Copilot, and others. If your team wants the "review intent before reviewing code" benefit without adopting a whole methodology, this is the closer fit for the Light Spec lane.

**[mattpocock/skills](https://github.com/mattpocock/skills)** takes a different angle entirely. Matt Pocock's own README calls out Spec-Kit by name: approaches that "own the process" can take away your control and make bugs in the process hard to resolve. His skills are small, composable, and meant to be picked individually rather than adopted as one framework: `/grill-me` for interrogating a plan before you build it, `/to-spec` for turning a conversation into a spec, `/to-tickets` for breaking that into tracer-bullet work items, and `/tdd` and `/code-review` for the build loop. That fits teams who already have their own change process (like the bank in this post) and just want a couple of sharp habits, not a new pipeline bolted onto the one they have.

Neither replaces the decision you still have to make: how much structure does this specific change need. They just give you a lower-ceremony way to answer "some" instead of jumping straight to "all of it."

---

## KPI check: is your process too heavy?

If the workflow is healthy, you usually see:

- lead time stable or improving,
- rework rate decreasing,
- escaped defects stable or improving.

If the workflow is too heavy, you usually see:

- time to first commit rising,
- review wait time rising,
- defect trend unchanged.

That pattern means you are paying more process cost without getting more quality.

---

## A 30-day reset if your team feels process drag

### Week 1

- classify incoming work as full/light/no-spec,
- stop forcing full-spec on low-risk items.

### Week 2

- introduce one-page light-spec template,
- reserve full-spec reviews for high-risk work.

### Week 3

- track three signals only: lead time, rework rate, escaped defects,
- compare against the previous month.

### Week 4

- keep what improved outcomes,
- remove what only increased ceremony.

You do not need a transformation program to do this.
You need one month of disciplined simplification.

---

## Failure modes worth avoiding

1. **Treating all work like architecture work**
2. **Confusing longer docs with better thinking**
3. **Freezing plans too early when reality changed**
4. **Never reviewing whether the process still pays off**

---

## The operating principle

The old failure mode was obvious: no structure, lots of rework.

The new failure mode is subtler: too much structure, slow delivery.

So the goal is not maximum process.
The goal is this:

> **Apply the minimum structure that reliably protects quality.**

That is how spec-driven development scales.

---

## Related reading

- [From Vibe Coding to Spec-Driven Development: Part 1](/from-vibe-coding-to-spec-driven-development)
- [The real cost of AI coding agents: what your team actually spends](/the-real-cost-of-ai-coding-agents)
- [AI coding agents need KPIs: how to measure speed, quality, reliability, and cost](/ai-coding-agents-need-kpis)
- [Single-agent, tools, or a team?](/single-agent-tools-or-a-team)

