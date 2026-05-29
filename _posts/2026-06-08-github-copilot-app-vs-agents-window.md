---
layout: post
title: "Two agent surfaces: GitHub Copilot App vs. VS Code Agents Window"
date: 2026-06-08 09:00:00 +0000
categories: [AI, Development]
tags: [github-copilot, ai-assisted-development, agents, vscode, developer-tools]
author: hidde
description: "GitHub shipped a standalone desktop app on May 14. Microsoft added an Agents Window to VS Code the same month. Both call themselves agent-first. Here is what they actually do and which one you should reach for."
image: '/images/agent.png'
featured: false
toc: true
---

VS Code now has an "Open in Agents" button in the title bar. Not a new sidebar panel. A whole separate window, with its own purpose: managing agent sessions instead of writing code.

On May 14, 2026, GitHub shipped a standalone desktop app aimed at the same thing: a dedicated surface for directing coding agents. Both teams arrived at the same conclusion in the same month. The chat panel was a starting point, not the destination.

---

## How we got here

Three milestones shaped how agent-first development ended up needing its own surface.

**March 2025.** VS Code v1.99 shipped agent mode to stable. It joined Ask and Edit in the chat panel, three tabs in a sidebar. Agent mode could read files, run terminal commands, and call MCP tools. The surface still said "assistant," but the capability was already pointing past that.

**May 2025.** VS Code v1.101 added tool sets and custom chat modes. The GitHub Pull Requests extension added Copilot coding agent integration: you could assign an issue to Copilot from inside VS Code and watch it work on a cloud-hosted branch.

**May 2026.** Two things shipped in the same month. GitHub released the Copilot App (v0.2.13, public tech preview, May 14), a standalone native desktop app built on the Copilot CLI. Microsoft added the Agents Window, a dedicated VS Code window mode for agent-first workflows. Same insight, two surfaces.

The shared logic: a sidebar next to your code is not the right place to manage work you are not doing yourself.

---

## GitHub Copilot App

The GitHub Copilot App is a native desktop application for Mac, Windows, and Linux. It is not a VS Code extension. It does not open your editor. It is distributed at [github.com/github/app](https://github.com/github/app) and reached v0.2.13 after 14 releases in its first two weeks.

**Availability:** Business and Enterprise accounts are getting access first. Pro and Pro+ users can join the early access waitlist. All usage draws from the same AI Credits pool that went live on June 1. (Full breakdown of what that costs in [The real cost of AI coding agents](/the-real-cost-of-ai-coding-agents.html).)

The app organizes work around three ideas.

### Start from GitHub context

Sessions launch from GitHub artifacts: an issue, a pull request, a saved prompt, or a previous session. The app shows an inbox of things that need attention across your repositories.

When a session starts from an issue, the agent already has the issue body, repo state, branch, and CI history. You are not assembling context manually. You are navigating it.

### Focused sessions

Each session gets its own isolated branch, worktree, file state, and conversation history. Pause at end of day, resume the next morning. Run three sessions across different repositories at the same time.

This matters because long agent tasks compete with your working environment inside an IDE. You edit one file, the agent edits another, and you are context-switching in both directions. Focused sessions remove that. The agent has its own space; you have yours.

### Steer, validate, ship

The app includes an integrated terminal and browser scoped to each session. You review the plan and the diff before anything merges.

The standout capability here is **Agent Merge**. After you approve the direction, the agent can automatically address pull request review comments, fix failing CI checks, and merge once your specified conditions are met.

In most current agent workflows, the last stretch is still manual. The agent finishes the implementation, a reviewer leaves a comment about error handling on line 42, and somebody has to pick that up. Agent Merge absorbs that step into the session. That changes the completion rate, not just the convenience.

---

## VS Code Agents Window

The Agents Window is a dedicated window mode in VS Code stable. Not a new product. Not a separate download. Open it with `code --agents`, the "Open in Agents" button in the VS Code title bar, or at `insiders.vscode.dev/agents` in any browser.

The framing VS Code uses is deliberate: the editor window is **code-first**, the Agents Window is **agent-first**. They share sessions, settings, and keybindings. Switching between them is switching windows, not switching tools.

### Sessions across workspaces

The Agents Window shows all your active agent sessions in one list, regardless of which workspace they belong to. Copilot CLI sessions, Copilot Cloud sessions, Claude agent sessions, all visible together, grouped by workspace. Pause and resume works the same way it does in the GitHub Copilot App.

### Worktree isolation

Each session gets its own git worktree. The agent's changes stay isolated until you decide what to do with them: commit, merge, or discard. Folder isolation is available as an alternative if worktrees do not fit the project setup.

Nothing the agent does inside a session touches your working tree until you bring it through.

### Changes panel

The Changes panel shows the full diff for a session. Inside the diff view, you can add inline feedback directly on specific lines without opening the chat. Write a comment on the change, and the agent picks it up in the next step. That is a more natural review gesture than switching to chat and re-describing what you noticed.

### Customizations panel

One panel manages everything that configures agent behavior: skills, instruction files, hooks, MCP servers, and plugins. If you have built a customization stack — the [SKILL.md files](https://hiddedesmet.com/skills-md-github-copilot), [AGENTS.md and .agent.md configuration](https://hiddedesmet.com/agent-md-explained), and the [five-file setup that ties them together](https://hiddedesmet.com/building-a-complete-agent-fleet) — the Customizations panel is the management UI for all of it. No digging through directory trees.

### Browser access and sub-sessions

`insiders.vscode.dev/agents` tunnels into your VS Code instance and gives you the full Agents Window from any browser on any device. A session running on your desktop is accessible from a tablet, a different machine, or a remote connection without losing state.

Sessions also support sub-sessions: parallel agent tasks scoped within the same workspace, coordinated from the same Agents Window.

---

## The comparison

Both surfaces do the same fundamental thing: move agent sessions out of the editor and into a dedicated context with isolation, state persistence, and a management interface. The differences come from what each one brings to that context.

| | GitHub Copilot App | VS Code Agents Window |
|---|---|---|
| **Surface type** | Standalone desktop app | VS Code window mode |
| **Built on** | Copilot CLI | VS Code + extensions |
| **Current status** | Public tech preview (v0.2.13) | Preview, available in stable |
| **Availability** | Business/Enterprise now; Pro/Pro+ waitlist | All VS Code users |
| **Requires VS Code** | No | Yes |
| **Session isolation** | Own branch + worktree per session | Git worktree per session |
| **GitHub integration** | Native (issues, PRs, inbox, CI status) | Via GitHub Pull Requests extension |
| **Agent Merge** | Yes | No |
| **Browser access** | No | insiders.vscode.dev/agents |
| **Code editing** | No | Full VS Code capabilities |
| **Customizations UI** | Not documented | Customizations panel (skills, MCP, hooks) |
| **Multiple repos simultaneously** | Yes | Grouped by workspace |

### When to use the GitHub Copilot App

You are directing work on multiple GitHub repositories and you are not writing the code yourself. Tasks start from issues or pull requests that already exist on GitHub. You want Agent Merge to handle the close: addressing review comments, fixing CI, merging when conditions are met. You do not need VS Code running.

### When to use the VS Code Agents Window

You run agents on local projects. You still write some code yourself. Your customization stack is built out and you want to manage it from one panel. Your sessions are scoped to repositories you actively work in.

### When to stay in the main editor window

You are writing code with AI assistance, not delegating tasks. Agent mode in the regular VS Code editor window is still the right surface for most day-to-day development work. These two new surfaces handle the subset where you are more manager than author.

---

## My take

The chat panel served its moment. It put the agent adjacent to your code because that was the only available surface in 2023. These two releases are the first honest answer to what "AI does the work, you direct it" actually requires: session isolation, statefulness, and a management surface that does not compete with your own editing.

The GitHub Copilot App has the stronger story for GitHub-native workflows. Agent Merge is the specific capability worth watching. The number of agent sessions that stall on a review comment or a small CI failure is high. Bringing that closing step inside the session changes the completion rate in a real way.

The VS Code Agents Window has the stronger story for daily development. It requires nothing new from your existing setup. The Customizations panel gives a management surface to the configuration work many VS Code users have already done. Browser access via dev tunnel is a quiet feature with a practical use: a long-running session stays accessible from anywhere without re-establishing context.

The two are not competing. They address the same shift from different angles. The GitHub Copilot App is for orchestrating work you are mostly not writing. The Agents Window is for developers who still write code but want agent work fully separated from it. Both get the core insight right: agent sessions are units of work with state, not conversations that disappear when you close a window.

---

## Get started

**Try `code --agents` today.** It is in VS Code stable. No additional extension required. Open a session and look at the Customizations panel — if you have instruction files or MCP servers configured, they are already listed there.

**Join the GitHub Copilot App waitlist.** Available at [github.com/github/app](https://github.com/github/app). Business and Enterprise accounts are rolling out now.

**Check your credits before you scale up.** Both surfaces consume AI Credits under GitHub's usage-based billing. [The per-model rates and full cost breakdown are here](/the-real-cost-of-ai-coding-agents.html).
