---
layout: post
title: "Single-agent, tools, or a team? A practical comparison of AI coding setups"
date: 2026-04-17
categories: [AI, Development]
tags: [ai-assisted-development, agents, multi-agent, mcp, github-copilot, claude, orchestration, prompt-engineering]
author: hidde
description: "Single-agent, agent-with-tools, or multi-agent? The same feature through all three setups, the failure modes to watch for, and a decision matrix you can actually use."
image: /images/agent.png
featured: false
toc: true
---

Every week someone asks me which agent setup they should use. Should they just chat with one model? Give it MCP tools? Or wire up a whole team of specialists with a planner, implementer, and reviewer?

The honest answer is that it depends on the job. But "it depends" is a lazy answer, so this post walks through the same realistic feature in all three setups, what each one tends to get right or wrong, and the decision framework I use to pick between them.

---

## The three archetypes

Before the comparison, clear definitions. These terms get used loosely, so here is what I mean by each one.

| Setup | What it is | Typical example |
|-------|-----------|-----------------|
| **Single-agent** | One model, one prompt, one context window. No external tools beyond what the IDE provides. | A chat session in Copilot or Claude where you paste code and iterate. |
| **Agent-with-tools** | One model that can call external tools through MCP or function calling. Still one context window, but it can read files, run commands, search docs, hit APIs. | Claude Code with a few MCP servers attached, or Copilot's agent mode with tool access. |
| **Multi-agent** | Multiple specialized agents coordinated by an orchestrator. Each agent has its own role, prompt, and often its own context window. | A planner agent writes a spec, an implementer agent writes code, a reviewer agent checks it. |

The line between "agent-with-tools" and "multi-agent" blurs in practice. If your single agent can spawn subagents, is that one agent or many? For this post, multi-agent means at least two agents with distinct roles that hand work to each other.

---

## The benchmark task

Same task, same model family, same spec. Different orchestration.

The task: add rate limiting to an ASP.NET Core Minimal API service inside a [.NET Aspire](https://aspire.dev/) solution. A handful of endpoints, a Redis integration already wired up in the AppHost, service defaults for OpenTelemetry and health checks in place, and an existing xUnit test project using `Aspire.Hosting.Testing`. No current rate-limit implementation.

The spec called for per-IP and per-API-key limits using the built-in `Microsoft.AspNetCore.RateLimiting` middleware, configurable windows via `appsettings.json`, proper 429 responses with `Retry-After` headers, and integration tests that spin the AppHost up with `DistributedApplicationTestingBuilder` and hit both limiters. The distributed story (sharing counters across instances via Redis) was explicitly called out as a follow-up, not part of this first pass, because ASP.NET Core's built-in rate limiter is in-process.

The model family was the Claude 4.x line, with Haiku used for worker roles in the multi-agent run. Here is what each setup tends to get right and wrong on a task like this.

### Single-agent

Fast and cheap, but it forgets Aspire conventions. The typical symptom is adding the rate limiter directly in `Program.cs` of the API project, instead of extending the shared `ServiceDefaults` extension methods where health checks and telemetry already live. The code works. It does not fit the shape of the solution. On a multi-service Aspire setup, that difference shows up the moment you add a second API and realise the limiter is not there.

### Agent-with-tools

Spends noticeable time reading the AppHost wiring and the `ServiceDefaults` project before writing anything. That reading time pays for itself. It finds the existing Redis integration registered in the AppHost, wires the connection through service discovery rather than hardcoding a string, and writes tests that actually boot the distributed application with `DistributedApplicationTestingBuilder`. Where it tends to trip: tool-call loops on edge cases like API-key header casing, where it retries the same fix with minor variations instead of stepping back.

### Multi-agent

Slower and more expensive, but cleaner on the first pass. A planner agent produces the spec, catches the header-case edge case, and explicitly places the rate-limit registration in `ServiceDefaults` so future Aspire services pick it up. An implementer works against that spec. A reviewer catches missing tests, typically around the `Retry-After` header or the 429 response shape. The trade-off is everything you do not see: the orchestration tokens, the handoff artifacts, the longer wall-clock time.

---

## The decision matrix

Across tasks like the one above, a pattern emerges. Here is the matrix I use now.

| Factor | Single-agent | Agent-with-tools | Multi-agent |
|--------|:-------------:|:----------------:|:-----------:|
| Task complexity | Low | Medium | High |
| Codebase size | Small | Medium to large | Medium to large |
| Context needed | Fits in one prompt | Needs file reads, search | Needs isolated reasoning per step |
| Determinism required | Low | Medium | High |
| Cost sensitivity | High | Medium | Low |
| Debuggability | Easy | Medium | Hard |
| Setup effort | None | Low | Medium to high |

Two rules of thumb I find useful:

1. If the task fits in a single prompt with its full context, do not reach for multi-agent. You will pay more for less.
2. If the task needs work that a human would split across multiple roles (design, implement, review), multi-agent usually wins, but only if the handoff artifacts are good.

That second rule matters more than people expect. Multi-agent systems live or die on what they pass between agents. A vague spec between a planner and an implementer is worse than a single agent with a clear prompt. Good specs, clear task lists, and structured progress logs are what make the whole thing work.

---

## Cost and latency, without the hype

Multi-agent looks expensive on paper, and often is, but model routing changes the picture.

A sensible split: orchestrator and planner on Sonnet, implementer on Sonnet (coding is where quality matters most), and reviewer, test-runner, and doc-updater roles on Haiku. At current [Anthropic pricing](https://www.anthropic.com/pricing), Haiku 4.5 sits at roughly a third of Sonnet 4.5's per-token cost, so offloading bounded, focused roles to Haiku is a real saving without a real quality hit on those roles.

Latency does not benefit from routing the same way. Multi-agent is sequential by nature, each agent waiting for the previous one's output. Some roles parallelize (test and lint agents can run side by side, doc updates can run after the implementer commits) but the planner-implementer-reviewer chain stays serial.

Rule of thumb: if you are running hundreds of these a day in CI, cost matters more than latency. If you are running a handful during an afternoon of focused work, latency matters more than cost.

---

## Failure modes

Every setup fails differently. Knowing how they fail tells you what guardrails to put in place.

### Single-agent failures

**Context collapse.** The model runs out of room and starts forgetting earlier parts of the conversation. You notice it when it reintroduces a bug you already fixed.

**Pattern invention.** Without tool access to read the codebase, the model guesses at conventions and often guesses wrong. The code compiles, but it does not fit.

**Silent scope creep.** Without explicit planning, the model quietly expands the task. You asked for rate limiting, you got rate limiting plus a new config system you never asked for.

### Agent-with-tools failures

**Tool-call loops.** The agent retries the same failing tool call with slight variations, burning tokens without making progress. The classic signature is three near-identical error messages in a row.

**Context bloat.** Every tool call adds output to the context. An aggressive file-reader can exhaust the context window on a medium codebase before it writes a single line.

**Over-reading.** The agent reads way more files than it needs because its prompt rewards thoroughness. You pay for exploration that did not change the answer.

### Multi-agent failures

**Lying handoffs.** Agent A reports a task as done, but it is not. Agent B trusts the handoff and builds on broken foundations. This one hurts because the failure shows up downstream.

**Coordination overhead.** The orchestrator spends more tokens deciding who does what than the workers spend actually working.

**Runaway cost.** Without a budget cap or iteration limit, a multi-agent loop can spiral. I have seen a misconfigured Ralph-style loop burn through a day's token budget in an hour.

**Debugging hell.** When something goes wrong, which agent caused it? Good logging and per-agent artifacts make this tractable. No logging makes it nearly impossible.

---

## When to pick what

The short version, with the caveats that matter.

### Pick single-agent when

- The task fits in one prompt with the full context you need
- You want fast feedback and low cost
- You are exploring, prototyping, or doing small edits
- You do not need the output to be production-ready without review

### Pick agent-with-tools when

- The task needs real codebase awareness (reading files, running tests, searching docs)
- You want one coherent reasoning thread but with access to external information
- The codebase is medium-sized and conventions matter
- You are the reviewer and will check the final result

### Pick multi-agent when

- The task has naturally separable roles (plan, implement, review, test)
- You need determinism and repeatability, like in CI or scheduled runs
- The context needed exceeds one window if you tried to do it single-agent
- You have good handoff artifacts (specs, task lists, progress logs)
- You can invest in logging and guardrails up front

The anti-pattern I see most often is reaching for multi-agent because it sounds sophisticated, for a task that a single well-prompted agent would have finished in two minutes. Complexity is not a feature.

---

## Where this is going

The boundary between these three archetypes is thinning. Modern agents spawn subagents mid-task. MCP servers wrap what used to be full multi-agent systems behind a single tool call. The question is shifting from "which setup do I pick" to "which shape does this specific task want right now".

My current working model: start with the simplest setup that might work, measure, and only add structure when the failure mode demands it. Context collapse means you need better context management, not always more agents. A pattern-invention problem means tool access, not always a reviewer agent. A determinism problem is usually where multi-agent pays off.

If you are starting fresh, here is the path I would take:

1. Start every task in a single-agent chat to feel the shape of the problem
2. Move to agent-with-tools the moment you need to read files or run commands
3. Reach for multi-agent only when the task clearly splits into distinct roles, or when you need the same workflow to run repeatedly in CI

The best AI coding setup is the smallest one that gets the job done. Everything above that is complexity you will have to debug later.
