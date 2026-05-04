---
layout: post
title: "Five files, one repo: the complete Copilot customization stack"
date: 2026-05-04
categories: [AI, Development]
tags: [github-copilot, agents-md, agent-md, skill-md, instructions-md, prompt-md, ai-assisted-development, dotnet-aspire, agent-customization, vscode]
author: hidde
description: "Five customization files, one .NET Aspire repo, one complete agent setup. How AGENTS.md, .instructions.md, SKILL.md, .prompt.md, and .agent.md work together in practice."
featured: true
toc: true
---

Over the last few posts I covered [SKILL.md](https://hiddedesmet.com/skills-md-github-copilot), [AGENTS.md and .agent.md](https://hiddedesmet.com/agent-md-explained), and [multi-agent decision frameworks](https://hiddedesmet.com/single-agent-tools-or-a-team). Every post handled one file type. Nobody showed what happens when you use all five at once.

This post puts them in one place: one .NET Aspire repo, five customization files, and a walkthrough of how they fire together on a real task.

The repo is a .NET Aspire solution with a Web API, a background worker, Redis wired up in the AppHost, OpenTelemetry through service defaults, and an xUnit test project using `Aspire.Hosting.Testing`. It has enough moving parts to show the interactions, but not so many that the article turns into a tour of unrelated code.

---

## The five layers

Five file types, five jobs. Here is the full stack for this project.

| Layer | File | Location | Role in this project |
|-------|------|----------|---------------------|
| Repo rules | `AGENTS.md` | root | Aspire conventions, commands, safety rules |
| Scoped instructions | `.instructions.md` | `.github/instructions/` | C#, Bicep, and test-specific rules |
| Skills | `SKILL.md` | `.github/skills/` | Aspire patterns, on-demand expertise |
| Prompt files | `.prompt.md` | `.github/prompts/` | Scaffold a service, generate a PR summary |
| Custom agents | `.agent.md` | `.github/agents/` | Planner, implementer, reviewer |

Each layer has a different loading behavior. Some are always on. Some auto-match. Some you invoke manually. That distinction is why I keep them separate instead of putting everything in one file.

---

## Layer 1: AGENTS.md: the repo rules

Covered in depth in [the AGENTS.md post](https://hiddedesmet.com/agent-md-explained). Quick version: `AGENTS.md` is always-on project guidance. It loads on every chat request when `chat.useAgentsMdFile` is enabled in VS Code.

For this Aspire solution:

```markdown
# AGENTS.md

## Project

This is a .NET Aspire solution. The AppHost is in `src/MyApp.AppHost/`.
Service projects live under `src/`. Generated output in `bin/` and `obj/`; never edit those directly.

## Commands

- Build: `dotnet build`
- Run: `dotnet run --project src/MyApp.AppHost`
- Test: `dotnet test`
- Lint: `dotnet format --verify-no-changes`

## Conventions

- Each service gets its own project under `src/`.
- Shared contracts live in `src/MyApp.Contracts/`.
- Use dependency injection; do not `new` up services manually.
- Configuration goes through Aspire service defaults, not hardcoded strings.
- All database access goes through repository classes.

## Safety

- Do not edit generated files in `bin/` or `obj/`.
- Do not hardcode connection strings or secrets; use Aspire service discovery and user-secrets.
- All new endpoints require input validation and auth middleware.
```

Nothing new here if you read the last post. The important part is what `AGENTS.md` does *not* do: it does not contain C#-specific style rules (those live in `.instructions.md`), Aspire testing patterns (that is a skill), or repeatable workflows (those are prompt files). It is the foundation layer. Everything else stacks on top.

---

## Layer 2: .instructions.md: scoped rules

This is the file type most people skip, and the one that solves the most annoying problem: your Bicep file getting C# advice because the global instructions tried to cover everything.

An `.instructions.md` file is a Markdown document with YAML frontmatter that includes an `applyTo` glob pattern. Copilot loads the instructions only when the active file matches the pattern. They live in `.github/instructions/`.

Three examples for this Aspire repo.

**C# conventions**: `.github/instructions/csharp.instructions.md`:

```markdown
---
applyTo: "**/*.cs"
---

- Use file-scoped namespaces.
- Prefer `record` types for DTOs and request/response models.
- Use primary constructors for dependency injection where the class has no other logic.
- Mark classes as `sealed` unless inheritance is intentional.
- Use `ILogger<T>` for logging; never `Console.WriteLine`.
- Async methods return `Task` or `Task<T>`, never `async void`.
- Name cancellation token parameters `cancellationToken`, not `ct` or `token`.
```

**Bicep conventions**: `.github/instructions/bicep.instructions.md`:

```markdown
---
applyTo: "**/*.bicep"
---

- Use `param` with `@description` decorators for every parameter.
- Environment-specific values go in `.bicepparam` files, not hardcoded.
- Name resources with the pattern `{workload}-{resource-type}-{environment}`.
- Use modules from `infra/modules/` for reusable components.
- Always set `location` from a parameter, never hardcode a region.
```

**Test conventions**: `.github/instructions/testing.instructions.md`:

```markdown
---
applyTo: "**/tests/**"
---

- Use xUnit with `Aspire.Hosting.Testing` for integration tests.
- Boot the full AppHost with `DistributedApplicationTestingBuilder` in integration tests.
- Use `Arrange-Act-Assert` structure with blank lines between sections.
- Name test methods `MethodName_Scenario_ExpectedResult`.
- Never share state between tests; each test gets a fresh AppHost instance.
- Use `WebApplicationFactory<T>` only for unit-level HTTP tests that do not need Aspire orchestration.
```

The `applyTo` pattern does the routing. When you are editing a `.cs` file, Copilot loads the C# instructions. When you open a `.bicep` file, it loads the Bicep instructions. When you are writing tests, it loads the testing rules. The rest stays out of context.

This is the same principle the [SKILL.md post](https://hiddedesmet.com/skills-md-github-copilot) described for skills: load only what fits the task. The difference is that `.instructions.md` files match on *file patterns*, while skills match on *task descriptions*. Instructions are about the file you are touching. Skills are about the problem you are solving.

A few practical notes:

- You can have as many `.instructions.md` files as you need. One per concern is cleaner than one giant file with sections.
- If two instruction files both match (e.g., a test file that is also a `.cs` file), both load. Keep them complementary, not contradictory.
- The `applyTo` pattern supports standard glob syntax. `**/*.cs` catches all C# files. `src/MyApp.Api/**` scopes to one project.
- Instructions do not create a dropdown item. They do not restrict tools. They are passive context, like `AGENTS.md`, but scoped.

If you already have a `copilot-instructions.md` that has grown into a 200-line monster, this is the migration path: break it into scoped `.instructions.md` files. The global file handles universal rules. Each scoped file handles language or domain-specific ones. Context stays lean.

---

## Layer 3: SKILL.md: on-demand expertise

Covered in depth in [the SKILL.md post](https://hiddedesmet.com/skills-md-github-copilot). Quick recap: a skill is a folder with a `SKILL.md` that loads automatically when the task description matches, and stays silent otherwise.

For this Aspire repo, I would add one skill first: Aspire integration testing patterns.

Save as `.github/skills/aspire-testing/SKILL.md`:

```markdown
---
name: aspire-testing
description: |
  Write and debug integration tests for .NET Aspire applications using
  Aspire.Hosting.Testing. USE FOR: DistributedApplicationTestingBuilder,
  AppHost test setup, resource readiness checks, service endpoint
  resolution in tests, health check verification.
  DO NOT USE FOR: unit tests without Aspire orchestration (use standard
  xUnit patterns), load testing, production health monitoring.
---

# Aspire Integration Testing

## Core pattern

Every integration test boots the full AppHost via
`DistributedApplicationTestingBuilder`. This gives you real service
discovery, real resource wiring, and real health checks.

​```csharp
public class ApiTests : IAsyncLifetime
{
    private DistributedApplication _app = null!;
    private HttpClient _client = null!;
    private readonly CancellationTokenSource _timeout = new(TimeSpan.FromSeconds(30));

    public async Task InitializeAsync()
    {
        var builder = await DistributedApplicationTestingBuilder
            .CreateAsync<Projects.MyApp_AppHost>();

        _app = await builder.BuildAsync();
        await _app.StartAsync();

        await _app.ResourceNotifications
            .WaitForResourceHealthyAsync("api", _timeout.Token);

        _client = _app.CreateHttpClient("api");
    }

    public async Task DisposeAsync()
    {
        _timeout.Dispose();
        await _app.DisposeAsync();
    }

    [Fact]
    public async Task HealthEndpoint_ReturnsHealthy()
    {
        var response = await _client.GetAsync("/health");
        response.EnsureSuccessStatusCode();
    }
}
​```

## Rules

1. Always call `await _app.StartAsync()` and wait for the resource to become healthy before making HTTP calls.
2. Use `_app.CreateHttpClient("resource-name")` to get a client
   that resolves through Aspire service discovery. Never hardcode ports.
3. Dispose the app after each test class to avoid port conflicts.
4. For tests that need Redis, the real Redis container spins up
   through the AppHost. Do not mock it unless you have a specific reason.
```

This skill loads when someone asks "how do I test my Aspire service" or "write an integration test for the health endpoint." It stays silent when someone asks about Bicep naming conventions. The `DO NOT USE FOR` line is the difference between a skill that helps and one that pollutes context on the wrong task.

---

## Layer 4: .prompt.md: repeatable workflows

Prompt files are the customization type that overlaps most with what people already do manually: copy-paste a prompt from Notion, adjust it for the current task, run it. A `.prompt.md` file turns that into something you invoke with a slash command.

They live in `.github/prompts/` and use YAML frontmatter with a `description` and an optional `agent`, `model`, and `tools` list. The body is the prompt template.

Two examples for this Aspire project.

**Scaffold a new service**: `.github/prompts/scaffold-aspire-service.prompt.md`:

```markdown
---
agent: 'agent'
description: "Scaffold a new Aspire service project with AppHost registration, service defaults, and a health check endpoint"
---

Create a new service project for this .NET Aspire solution. The service name is: ${input:serviceName}

Steps:

1. Create a new project at `src/MyApp.${input:serviceName}/` using the ASP.NET Core Web API template.
2. Add a project reference to `src/MyApp.ServiceDefaults/`.
3. In `Program.cs`, call `builder.AddServiceDefaults()` and `app.MapDefaultEndpoints()`.
4. Register the service in the AppHost (`src/MyApp.AppHost/Program.cs`) with `builder.AddProject<Projects.MyApp_${input:serviceName}>("${input:serviceName}")`.
5. Add a minimal health check endpoint at `/health`.
6. Create a test class in `tests/MyApp.Tests/` that boots the AppHost and verifies the new service responds to `/health`.

Follow the conventions in AGENTS.md. Use the patterns from the existing API service as a reference.
```

**Generate a PR summary**: `.github/prompts/pr-summary.prompt.md`:

```markdown
---
agent: 'agent'
description: "Generate a PR description from the current branch diff"
---

Analyze the changes on the current branch compared to `main`.

1. Run `git diff main...HEAD --stat` to get the file list.
2. Run `git diff main...HEAD` to get the full diff.
3. Read any modified or new test files.

Write a PR description with:

- **Summary**: 2-3 sentences on what changed and why.
- **Changes**: bullet list grouped by area (API, worker, infra, tests).
- **Testing**: what was tested, how to verify manually.
- **Notes**: anything a reviewer should pay extra attention to.

Use the existing PR convention from AGENTS.md.
```

Invoke either one from the Copilot chat with the file name as a slash command.

The distinction from the other file types matters:

| File type | How it activates | What it does |
|-----------|-----------------|--------------|
| `AGENTS.md` | Always on | Passive context |
| `.instructions.md` | Auto-matches file pattern | Passive context, scoped |
| `SKILL.md` | Auto-matches task description | Passive expertise, on demand |
| **`.prompt.md`** | **Manual invocation** | **Active workflow, runs a task** |
| `.agent.md` | Manual selection | Persistent persona with tool/model config |

Skills load knowledge. Prompt files run workflows. A skill teaches Copilot how Aspire testing works. A prompt file tells Copilot to scaffold a service right now, with specific steps, and produce specific output.

If you find yourself pasting the same 15-line prompt from a shared doc more than twice a week, that is a prompt file.

---

## Layer 5: .agent.md: specialist roles

Covered in depth in [the AGENTS.md post](https://hiddedesmet.com/agent-md-explained). For this repo, three custom agents that form a handoff chain.

**Planner**: `.github/agents/planner.agent.md`:

```markdown
---
name: planner
description: Creates implementation plans for features and refactoring tasks
tools: ['search/codebase', 'search/usages']
handoffs:
  - label: Start Implementation
    agent: implementer
    prompt: Implement the plan above. Work through each phase in order.
    send: false
---

You are a technical planner for a .NET Aspire solution.

When asked to plan work:

1. Read the relevant files and understand the existing patterns.
2. Identify what needs to change and what should stay the same.
3. Break the work into small phases with clear deliverables.
4. For each phase, list: files to change, tests to add, risks.
5. Do not modify any code. Planning only.
```

**Implementer**: `.github/agents/implementer.agent.md`:

```markdown
---
name: implementer
description: Implements features following a plan, writing code and tests
tools: ['search/codebase', 'edit', 'execute/runInTerminal']
handoffs:
  - label: Review Changes
    agent: reviewer
    prompt: Review the implementation above against the plan and project conventions.
    send: false
---

You are an implementer for a .NET Aspire solution.

When implementing:

1. Follow the plan exactly. Do not add scope.
2. Write the implementation first, then the tests.
3. Run `dotnet build` after changes to verify compilation.
4. Run `dotnet test` after adding tests to verify they pass.
5. Commit with a conventional commit message.
```

**Reviewer**: `.github/agents/reviewer.agent.md`:

```markdown
---
name: reviewer
description: Reviews code for quality, security, and convention adherence
tools: ['search/codebase', 'search/usages']
---

You are a code reviewer for a .NET Aspire solution.

When reviewing:

1. Check that the implementation matches the plan.
2. Verify input validation on all new endpoints.
3. Confirm auth middleware is applied where required.
4. Look for hardcoded strings that should use service discovery.
5. Check that tests cover the new behavior.
6. Flag missing error handling and inconsistent patterns.

Return findings grouped by severity: critical, warning, suggestion.
Do not modify files.
```

Notice the tool restrictions. The planner is read-only. The implementer can search, edit, and run terminal commands. The reviewer is read-only again. Tool names are environment-specific, so use the names VS Code shows in your tool picker rather than treating this list as a universal schema. That is safer than telling one general-purpose agent to "be careful" and hoping for the best.

The handoff chain creates a controlled flow: planner produces a plan, offers a button to hand off to the implementer, and the implementer offers a button to hand off to the reviewer. Each role gets its own context and constraints. The [multi-agent post](https://hiddedesmet.com/single-agent-tools-or-a-team) covers when this pattern is worth the overhead and when a single well-prompted agent is enough.

---

## How the layers interact

Here is what happens when all five files are present. Walk through one task: "add a health check endpoint to the Web API."

| Step | What happens | Which layer fires |
|------|-------------|-------------------|
| 1 | You open Copilot chat in the repo | `AGENTS.md` loads, so Copilot knows this is an Aspire solution, knows the commands, and knows the conventions |
| 2 | You switch to the planner agent | `.agent.md` activates the planner persona and restricts tools to read-only |
| 3 | Planner reads the worker project (`*.cs` files) | `.instructions.md` for `**/*.cs` loads the C# conventions |
| 4 | Planner references Aspire service defaults | `SKILL.md` auto-matches because the task involves Aspire patterns |
| 5 | Planner produces a plan and shows "Start Implementation" | Handoff button from `.agent.md` |
| 6 | You click the button, implementer agent takes over | New `.agent.md` persona, full tool access, follows the plan |
| 7 | Implementer writes code, runs `dotnet build`, runs `dotnet test` | `AGENTS.md` commands, `.instructions.md` C# and test rules, Aspire skill all in context |
| 8 | Implementer shows "Review Changes" button | Handoff to reviewer |
| 9 | After review, you run the PR summary prompt | `.prompt.md` runs manually and generates the PR description from the diff |

Nine steps. Five file types. Each one loaded at the right moment with the right scope. No single file tried to do everything.

The messy version is one 400-line `copilot-instructions.md` that loads Aspire patterns, C# conventions, Bicep rules, test strategies, PR templates, and agent behavior instructions on every single request, including the one where you just want to rename a variable. That file can cost 4,000+ tokens per request and contradict itself whenever two conventions conflict.

Five focused files keep context smaller and maintenance easier because each one has exactly one job.

---

## What to skip

The anti-patterns I see most often when people set this up:

**Building a fleet of 15 agents on day one.** You will use two of them. Start with one reviewer. Add agents when a workflow pain demands a specific role, not before.

**Duplicating `AGENTS.md` content inside every `.agent.md`.** If every agent needs to know it, `AGENTS.md` handles it once. Agent files define the role, not the repo.

**Making skills for things `.editorconfig` already enforces.** "Use 4-space indentation" is not a skill. It is an editor setting. Skills are for expertise that a configuration file cannot express.

**Using `.instructions.md` for rules that apply to every file type.** That is what `AGENTS.md` is for. Scoped instructions are for the rules that differ between file types. If a rule applies everywhere, it should not need a glob pattern.

**Putting workflow steps in a skill instead of a prompt file.** If it runs once and produces output, it is a prompt file. If it loads knowledge for an ongoing conversation, it is a skill. "Scaffold a service" is a prompt. "How Aspire service discovery works" is a skill.

---

## The starting kit

If I were adding all five layers to a repo from scratch, I would start with exactly this:

1. **`AGENTS.md`** at the repo root. Project context, commands, conventions, safety rules. Ten minutes to write, saves hours of re-explaining.

2. **One `.instructions.md`** scoped to your test files. Testing conventions are the rules that drift fastest across a team. Scoping them to `**/tests/**` keeps them out of production code context.

3. **One `SKILL.md`** for whatever piece of knowledge only two people on the team have. The Aspire testing pattern. The KQL query conventions. The Bicep module structure. The thing people keep asking about in Slack.

4. **One `.prompt.md`** for the task you repeat every week. PR summaries. Service scaffolding. Release notes. Whatever you copy-paste from a shared doc regularly.

5. **One `.agent.md`** for a reviewer role. Read-only tools, specific review criteria, structured output format. The role you would benefit from most when it is 11 PM and you are about to merge your own PR.

That is five files. Each one takes ten minutes to write. You can add more when the workflow tells you to, not when the documentation tells you that you should.

The principle: add the next file when a real pain demands it, not before. Complexity in your agent setup is no different from complexity in your code. It has to earn its place.
