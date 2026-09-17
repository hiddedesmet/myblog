---
layout: post
title: "Stop blaming the model: is your repository ready for AI agents?"
date: 2026-09-17 09:00:00 +0000
categories: [AI, Development, DevOps]
tags: [ai-coding-agents, github-copilot, repository-readiness, testing, ci-cd, developer-experience, devcontainers]
author: hidde
description: "A practical 20-point scorecard to test whether your repository gives AI coding agents the setup, context, tests, CI, and guardrails required to succeed."
featured: true
image: /images/stopblamingthemodel.png
toc: true
---

```text
# Repository A
$ npm test
Error: connect ECONNREFUSED 127.0.0.1:5432

# Repository B
$ make bootstrap
Environment ready.

$ make test-unit TEST=users
42 tests passed.
```

These are illustrative outputs, not benchmark results. Imagine giving the same AI coding agent the same bug in these two repositories.

In Repository A, it first has to discover which runtime to install, where the database comes from, which environment variables are required, and whether `npm test` even matches CI. Some of that knowledge exists, but only on a maintainer's laptop or in an old chat thread.

In Repository B, setup is executable. The toolchain is pinned. Tests bring their own fixtures. One command produces a useful verdict.

That is not a model comparison. It is a repository comparison.

Sometimes the model really is the problem: it can misunderstand the requirement or produce the wrong fix in a well-prepared repository. But changing it will not grant access to a private package feed. An agent may repair setup, but that consumes the session you meant to spend on the bug.

Before you blame the agent, test the environment you gave it.

This guide helps you find what is blocking useful delegation, decide which tasks you can safely hand over, and identify what to fix first. Start with a clean-room drill, use the six gates to diagnose the failures, then use the [20-point scorecard](#score-the-repository-out-of-20) to organize the repairs. The principles apply across coding agents; the implementation notes use GitHub Copilot cloud agent.

---

## Run the clean-room drill

Pick a small, reversible maintenance task, not an authentication change or cross-service redesign. Use your existing development and CI tools. If you use hosted compute, check the costs and plan requirements first.

Before any run with secrets or internal network access, restrict credentials, network access, and tools to what the task needs. [Gate five](#gate-five-autonomy-must-stop-at-the-security-boundary) covers those controls.

1. Start in a disposable environment with a clean checkout. No personal dotfiles, cached personal credentials, or manually started services. A fresh clone on your usual laptop does not count.
2. Bootstrap using committed setup guidance. Provision dedicated credentials separately; never commit secret values.
3. Run the focused test for the affected component.
4. Make the small change.
5. Run the complete pre-PR check.
6. Open a draft pull request with the validation results.

**Record every undocumented human intervention.** If a prerequisite blocks the drill, record it rather than silently repairing the environment and calling the run a success.

Record specific failures rather than just a pass/fail verdict:

```text
- runtime version had to be guessed
- package registry was undocumented
- test fixture existed only on one laptop
- lint worked only through the IDE
- CI ran an extra generated-code check
- credential permissions exceeded the task
- changed path had no owner
```

No cloud agent yet? Run the drill manually to check whether the committed setup instructions are enough.

<details class="post__details" markdown="1">
<summary>Copilot implementation note: checking the PR handoff</summary>

By default, Actions workflows wait for approval when Copilot pushes changes to a pull request. Inspect the proposed code, especially workflow changes, before selecting **Approve and run workflows**. Administrators can [disable this approval requirement](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings), but doing so can expose Actions secrets and write permissions to unreviewed code. Confirm the expected checks actually run.

When the PR is ready, mark it ready for review and check owner routing. GitHub does not automatically request code-owner reviews on drafts. Review requests alone do not block merges: verify the enforced checks and review requirements described in [gate four](#gate-four-ci-must-enforce-the-same-definition-of-done).

</details>

Use the six gates below to connect each failure to a repair. Repository instructions can point to the right commands; the drill tests whether those commands actually work.

---

## Gate one: a clean checkout must become a working environment

GitHub Copilot cloud agent starts in an ephemeral, GitHub Actions-powered environment. It does not inherit your laptop's cached credentials or manually installed services. A local agent may borrow those without exposing the setup gaps.

Commit enough information to select the tools and set up the project. Depending on the stack, that includes:

```text
.tool-versions / .nvmrc / global.json
package-lock.json / pnpm-lock.yaml / poetry.lock / Gemfile.lock
Makefile / justfile / scripts/bootstrap
.env.example
compose.yaml
.github/workflows/copilot-setup-steps.yml
.devcontainer/devcontainer.json
```

Not every repository needs every file. Use a dev container if it helps make setup repeatable. Version files and lockfiles help only if setup uses them and packages remain accessible.

Bootstrap should stop at a missing runtime, failed migration, or inaccessible registry and report what failed.

Commit safe defaults or an `.env.example`; inject secrets separately. Supply disposable services and representative fixtures. The agent should not need to invent a database just to test a validation rule.

{: .warning }
Copilot skips remaining setup steps after a non-zero exit code **but still starts the agent**. A failed bootstrap does not stop the session. Inspect the setup logs and make validation reject missing prerequisites.

<details class="post__details" markdown="1">
<summary>Copilot implementation note: setup, runners, and session limits</summary>

GitHub's [setup workflow](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment) lives at `.github/workflows/copilot-setup-steps.yml` on the default branch, with one job named `copilot-setup-steps`. It runs before the agent starts. Call shared project scripts from it so developers, CI, and agents use the same setup path.

GitHub also supports self-hosted runners and recommends ephemeral, single-use instances. Its [documented workflow limits](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent#limitations-of-copilot-cloud-agent) allow changes in one repository and branch at a time, at most one pull request per task, and 59 minutes per session. Research-only sessions need not open a pull request.

The one-repository limit does not stop an MCP server from reaching other repositories or systems. That access depends on the server's credentials and enabled tools, not the scope of the coding task.

</details>

---

## Gate two: give developers, agents, and CI the same commands

The build command is in `README.md`, integration tests are in a wiki, and generated-code validation exists only in CI. Searching for each command wastes time and makes it easy to miss a check.

Expose a small set of shared commands:

```text
make bootstrap     # prepare a clean environment
make build         # compile or package the project
make test-unit     # run the fast test layer
make lint          # run static checks
make generate      # refresh generated artifacts
make check         # run the complete pre-PR validation
```

These are example `Makefile` targets, not built-in commands. Your target must also implement `TEST=users` where used. Pick names that suit your project.

For a .NET repository, the same interface might use `dotnet restore`, `dotnet build`, `dotnet test`, and a committed PowerShell script such as `./scripts/check.ps1` for the full pre-PR check. npm scripts work too. The shared interface matters, not Make.

Check that they do what CI expects:

<div class="table-container" role="region" aria-label="Local commands compared with CI" tabindex="0" markdown="1">

| Repository says | CI actually does | Result |
|---|---|---|
| `npm test` | Tests plus schema generation | Stale generated files fail CI |
| Use the default runtime | Uses a pinned runtime | Local success hides CI failure |
| Run the linter | Treats warnings as errors | Avoidable review iteration |
| Start any database | Uses a specific service version | Integration behavior differs |

</div>

Put that index in a root `AGENTS.md` for agents that support it, or in `.github/copilot-instructions.md` for Copilot. Copilot cloud agent supports both. Include the shared commands, a short component map, generated files and how to regenerate them, owners of sensitive code, and slow or external checks.

For example: "User validation lives in `src/users/`; run `make test-unit TEST=users`. API clients are generated; refresh them with `make generate`, do not edit them by hand."

Keep secrets, one-off task requirements, and pages of copied architecture documentation out. Link to maintained docs and scripts instead of duplicating their contents. The [AGENTS.md guide](/agent-md-explained) covers the file choices; here, the test is whether the index gets the agent to the right code and command.

---

## Gate three: let the agent run focused tests

An agent changes validation logic in `src/users/`. Its only test command launches every browser journey, rebuilds three containers, and waits for a shared environment.

Let it check the affected component first:

```bash
make test-unit TEST=users
```

There is no universal two-minute rule. The smallest relevant check should run unattended, return meaningful exit codes, use deterministic fixtures, and work regardless of test order. Failures should explain what broke.

Three details make that feedback usable:

- **Make the focused command discoverable.** Put the component-to-test mapping in the repository instructions or link to it there. Document the filter syntax and make an empty test selection fail rather than look like success.
- **Separate flaky tests from reliable checks.** Track confirmed flakes in a visible quarantine with an owner and a repair issue. Do not retry until green and call it proof. Keep quarantined failures visible in CI, and record any coverage gap before delegating work that relies on those tests.
- **Keep the useful output short.** Show the failing test, expected and actual values, and the relevant stack trace first. Save verbose logs as artifacts. A wall of setup logs consumes context and can bury the error the agent needs to fix.

Follow with lint and type checks, then broader integration and end-to-end tests. Repository B catches the cheap failures first. Repository A is still looking for the database.

---

## Gate four: CI must enforce the same definition of done

Call the shared validation command from CI after checkout and setup:

```yaml
- name: Validate
  run: make check
```

Runtimes, services, permissions, and the tested commit must still agree. Document any checks that can only run in CI.

Require checks and human approval before merge. They cannot catch every defect, but they should block changes that fail the configured requirements. `CODEOWNERS` routes requests; to require an owner's approval, enable **Require review from Code Owners**.

<details class="post__details" markdown="1">
<summary>GitHub implementation note: rulesets and code owners</summary>

[Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) can require checks, pull requests, approvals, and configured code scanning results. Verify plan and repository availability, activate rules on the target branch, and audit bypass permissions.

Example `.github/CODEOWNERS`:

```text
/.github/CODEOWNERS  @your-org/platform-team
/.github/workflows/  @your-org/platform-team
/infra/              @your-org/platform-team @your-org/security-team
/src/auth/           @your-org/identity-team
```

Use real, visible teams with explicit write access. Standard code-owner review accepts **either** listed `/infra/` owner, not both. For both approvals, configure separate required-team reviews in a ruleset.

Protect `CODEOWNERS` itself. GitHub reads it from the pull request's **base branch**: edits cannot change that request's routing, but affect future requests once merged.

</details>

---

## Gate five: autonomy must stop at the security boundary

Making setup easy does not mean giving the agent the credentials from your laptop.

```text
Needed: download packages from a private registry
Grant:  read-only package access for this repository
Avoid:  a personal token with repository administration rights
```

Ordinary Copilot Agents secrets are available to the agent and setup scripts as environment variables. Calling something a secret does not hide it from the process using it.

The hosted firewall is **not a complete sandbox**: setup and MCP processes are outside its direct coverage, and GitHub documents potential bypasses. MCP tools can run without per-call approval. Review before merge cannot undo an external action already taken by a tool.

Before execution, configure and test:

- dedicated, least-privilege credentials,
- no production credentials for normal coding tasks,
- approved network destinations,
- isolated execution where practical.

Keep required review and ownership for sensitive paths, as described above. Neither replaces limits on what the agent can do during a session.

<details class="post__details" markdown="1">
<summary>Copilot implementation note: secrets, firewall, and MCP</summary>

- [Agents secrets and variables](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/configure-secrets-and-variables) are separate from Actions, Codespaces, and Dependabot. Restrict repository access and credential permissions. `COPILOT_MCP_` names are reserved for MCP servers.
- The [hosted firewall](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-the-firewall) covers processes started through the agent's Bash tool. Review the default dependency allowlist and additions. Do not disable it without replacement controls. Self-hosted runners and Windows need separately configured network controls; the integrated firewall is incompatible with them.
- Audit MCP credentials separately and [allowlist specific read-only tools](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers) where possible.

</details>

GitHub's [responsible-use guidance](https://docs.github.com/en/copilot/responsible-use/agents) also calls for review and testing: generated code can be inaccurate or insecure.

---

## Gate six: specify the task and its limits

Compare these two issues:

```text
Improve user validation.
```

```text
Reject an empty display name in src/users/.
Return the existing validation error type.
Add unit tests beside the current user-service tests.
Do not change the public API schema.
```

The second gives the agent a starting point and the reviewer acceptance criteria. GitHub [recommends this level of clarity](https://docs.github.com/en/copilot/tutorials/cloud-agent/get-the-best-results), with more human involvement for ambiguous, sensitive, or cross-repository work.

Use an issue form to ask for these details:

```text
Problem:
Acceptance criteria:
Expected validation:
Likely component or path:
Out of scope:
```

If the task depends on repositories the agent cannot access, undocumented business rules, or checks it cannot run, a maintainer needs to resolve those gaps or handle that part of the work.

---

## Score the repository out of 20

Use the failures from your drill as evidence, not the presence of a configuration file. The ten items below break the six gates into checks you can score separately. This is my suggested scorecard, not an industry benchmark. Score each item from 0 to 2:

- **0: Missing.** Absent, unknown, or dependent on undocumented knowledge.
- **1: Partial.** Documented or automated in places, but incomplete or dependent on an already-configured developer environment.
- **2: Proven.** Works from a clean environment and is repeatable or enforced.

*On small screens, scroll the scorecard sideways to compare all three ratings.*

<div class="table-container post__scorecard" role="region" aria-label="Repository readiness scorecard" tabindex="0" markdown="1">

| # | Gate | Criterion | 0 points | 1 point | 2 points |
|---:|---|---|---|---|---|
| 1 | 1: Environment | Clean bootstrap | No reliable path | Needs manual repair | Documented path works from clean checkout |
| 2 | 1: Environment | Pinned tools and dependencies | Versions guessed | Partly pinned | Required versions declared and reproducible |
| 3 | 1: Environment | Services, config, test data | Undocumented | Examples; manual setup | Safe config, services, fixtures automated |
| 4 | 2: Commands and context | Canonical commands | Scattered | Documented but inconsistent | Stable setup, build, test, lint, check interface |
| 5 | 2: Commands and context | Repository map | Structure inferred | Main layout described | Components, owners, generated files and regeneration paths maintained |
| 6 | 3: Focused tests | Focused validation | No practical proof | Slow, flaky, IDE-bound | Unattended, deterministic, targeted checks |
| 7 | 4: Merge gates | CI parity and enforcement | Missing or unrelated | Parity or enforcement gaps | Shared scripts; required checks block merge |
| 8 | 4: Merge gates | Review and ownership | No clear reviewer | Advisory only | Human review enforced; sensitive paths require owners or teams |
| 9 | 6: Task scope | Task contract | Vague issues | Some context captured | Outcome, validation, scope, exclusions required |
| 10 | 5: Security | Security boundary | Broad credentials or unrestricted access | Partial controls | Tested credential, network, tool, isolation controls; review enforced |

</div>

### Check the hard stops before interpreting the total

A high total cannot compensate for a missing prerequisite. Any of these conditions overrides the total:

1. **Bootstrap or focused validation is 0:** do not call the repository agent-ready.
2. **CI parity and enforcement or review controls is 0:** fix the merge gates before treating delegated changes as ready to merge. A nonzero score alone is not permission to enable auto-merge.
3. **The security boundary is 0:** do not provide autonomous execution with secrets or internal network access.

The two highest bands require **enforced checks and human review**, including required owners or teams for sensitive paths. Without those controls, do not exceed **Supervised only**, regardless of total.

### Use the score to organize the remaining improvements

<div class="table-container" role="region" aria-label="Readiness score bands" tabindex="0" markdown="1">

| Score | Interpretation | Recommended use |
|---:|---|---|
| **0–7** | Model-blame magnet | Fix setup and validation before judging agent performance |
| **8–13** | Supervised only | Use draft-PR experiments with close human steering |
| **14–17** | Ready for bounded tasks | Delegate small bugs, tests, docs, and contained maintenance |
| **18–20** | Strong agent foundation | Expand task classes gradually; retain required checks and review |

</div>

The ranges are recommendations, not measured success probabilities. A score of 18 does not guarantee a good pull request.

### Worked example: Repository A and Repository B

The opening outputs cannot establish ten scores. To show how the scorecard works, extend those fictional repositories with the assumptions below. These are illustrative drill findings, not measured results.

<div class="table-container" role="region" aria-label="Illustrative repository scores" tabindex="0" markdown="1">

| # | Criterion | A | B | Assumed finding |
|---:|---|---:|---:|---|
| 1 | Clean bootstrap | 0 | 2 | A has no documented working setup; B repeats it cleanly |
| 2 | Pinned tools and dependencies | 0 | 2 | A guesses versions; B installs declared versions |
| 3 | Services, config, test data | 0 | 2 | A relies on laptop state; B provisions safe fixtures and services |
| 4 | Canonical commands | 1 | 1 | Both document commands, but broader checks still differ from CI |
| 5 | Repository map | 1 | 1 | Both describe the layout but omit generated-file guidance |
| 6 | Focused validation | 0 | 2 | A cannot validate this task; B repeats targeted tests unattended |
| 7 | CI parity and enforcement | 1 | 1 | Both run CI, but not every required check blocks merging |
| 8 | Review and ownership | 1 | 1 | Both route reviews without enforcing owner approval |
| 9 | Task contract | 1 | 1 | Both capture the problem but leave scope exclusions optional |
| 10 | Security boundary | 0 | 1 | A has unrestricted access; B restricts credentials but has not tested all tool and network limits |
| | **Total** | **5/20** | **14/20** | **Apply the hard stops before assigning a verdict** |

</div>

**A is not agent-ready.** Bootstrap and focused validation are both 0. Its security score also rules out autonomous execution with secrets or internal access.

**B is still supervised only, despite scoring 14.** It loses six points across commands, context, CI, review, task scope, and security. Its unenforced merge controls cap the verdict. Fix those before calling it ready for bounded delegation; passing 42 tests does not settle the other criteria.

<details class="post__details" markdown="1">
<summary>Copyable Markdown scorecard for your next drill</summary>

Copy this into an issue and use the rating definitions above. Check a box when you have recorded a score and evidence, not merely found a configuration file.

```markdown
## Repository readiness drill
Task / commit:
Disposable environment:
Date / assessor:

Score each item 0, 1, or 2; add evidence and the next repair.
- [ ] 1. Clean bootstrap (gate 1):
- [ ] 2. Pinned tools and dependencies (gate 1):
- [ ] 3. Services, config, test data (gate 1):
- [ ] 4. Canonical commands (gate 2):
- [ ] 5. Repository map (gate 2):
- [ ] 6. Focused validation (gate 3):
- [ ] 7. CI parity and enforcement (gate 4):
- [ ] 8. Review and ownership (gate 4):
- [ ] 9. Task contract (gate 6):
- [ ] 10. Security boundary (gate 5):

Total: /20
Undocumented human interventions:
Hard stops: 1 or 6 = 0 means not agent-ready; 7 or 8 = 0
means fix merge gates; 10 = 0 means no autonomous execution
with secrets or internal access.
Without enforced checks and human review (including required
owners/teams for sensitive paths), cap at Supervised only.
Verdict after hard stops and review cap:
First repair / owner:
Repeat-drill result:
```

</details>

---

## Fix the first failure, then repeat the drill

Repository A needs a working validation path before its next score means much. Investigate the service requirement and make the needed service and fixtures reproducible. If the focused test does not need the database, remove that dependency. Then repeat the same bounded task from a clean environment.

Repository B needs to close the gaps beyond its focused tests: align the broader checks, enforce review, and verify access controls. Another green unit-test run will not earn those missing points.

For your repository, set access restrictions first, then repair bootstrap, focused tests, and CI enforcement in that order. Keep the fixes in shared tooling so the next developer benefits too. Once the drill runs without undocumented help, use the [AI coding agent KPI scorecard](/ai-coding-agents-need-kpis) to measure delivery outcomes separately from setup failures.

Pick one small task. Run the drill, record the first blocker, fix it, and repeat. A failed bootstrap and a wrong fix are different failures. Separate them before deciding what to change.

---

## Related reading

- [AGENTS.md vs .agent.md: repo rules and custom agent roles explained](/agent-md-explained)
- [Five files, one repo: the complete Copilot customization stack](/building-a-complete-agent-fleet)
- [AI coding agents need KPIs: how to measure speed, quality, reliability, and cost](/ai-coding-agents-need-kpis)
- [Terraform on Azure with guardrails: pre-commit, Trivy, and Anton Babenko's hooks](/terraform-azure-precommit-guardrails)

---

## Sources

Product behavior checked against the documentation below on September 17, 2026. The gates, score thresholds, and suggested fixes are editorial recommendations.

- [About GitHub Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent)
- [Best practices for using GitHub Copilot to work on tasks](https://docs.github.com/en/copilot/tutorials/cloud-agent/get-the-best-results)
- [Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/add-custom-instructions/add-repository-instructions)
- [Configuring settings for GitHub Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings)
- [Configuring the development environment for Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment)
- [Configuring secrets and variables for Copilot cloud agent](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/configure-secrets-and-variables)
- [Configure MCP servers for your repository](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/configure-mcp-servers)
- [Customizing or disabling the firewall for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-the-firewall)
- [Responsible use of GitHub Copilot agents](https://docs.github.com/en/copilot/responsible-use/agents)
- [Available rules for GitHub rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)
- [About code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)