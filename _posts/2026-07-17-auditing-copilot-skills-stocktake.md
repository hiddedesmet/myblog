---
layout: post
title: "105 skill folders, three coding agents, seven that don't exist: auditing your own SKILL.md library"
date: 2026-07-17
tags: [github-copilot, claude-code, skill-md, agent-customization, ai-assisted-development, developer-tools, maintenance]
author: hidde
description: "I counted the skill folders across Claude Code, a shared agents catalog, and Copilot on my own machine. 105 total, seven of them pointing at nothing. Here's what a real audit finds and the checklist to run it yourself."
image: /images/auditskills.png
featured: true
toc: true
---

Three coding agents run on this machine: Claude Code, GitHub Copilot in VS Code, and a shared `.agents/skills/` catalog a couple of other tools read from. That's three separate skill folders. I counted them while writing this post: `~/.claude/skills/` has 63 folders, `~/.agents/skills/` has 36, `~/.copilot/skills/` has 6. 105 total.

Seven of those folders, the entire `google-agents-cli-*` set, exist under the exact same names in both `~/.claude/skills/` and `~/.agents/skills/`. I went to open one for this post, expecting a Google Agent CLI skill. Instead: an empty `references/` folder and no `SKILL.md` at all. Not in either copy.

```text
$ ls ~/.claude/skills/google-agents-cli-eval/
references/

$ cat ~/.claude/skills/google-agents-cli-eval/SKILL.md
cat: SKILL.md: No such file or directory
```

Two tools, two folders, both pointed at a name that can never load, because the file it's supposed to read doesn't exist. That's what a year of adding skills without ever subtracting one looks like.

---

## Nothing tells you this is happening

I've written about this stack from the build side three times: [SKILL.md itself](https://hiddedesmet.com/skills-md-github-copilot), [AGENTS.md and custom agents](https://hiddedesmet.com/agent-md-explained), and [all five customization files working together](https://hiddedesmet.com/building-a-complete-agent-fleet). Every one of those posts is about adding a file, writing a good description, getting the loading behavior right.

None of them cover what happens after month twelve, when you have 105 skill folders and no idea which ones are dead weight.

And the reason it sneaks up on you is structural: [the loading loop for skills is index → match → load](https://hiddedesmet.com/skills-md-github-copilot#how-the-loading-loop-actually-works). A broken or redundant skill just sits in the index step. It costs a few tokens of frontmatter on every request, never matches, never loads, never throws an error. There's no CI job for "does this SKILL.md still make sense." No linter flags a skill that duplicates another one. No runtime exception fires when the folder is empty. The failure mode is silent by design, which is exactly why it needs a deliberate audit instead of waiting for something to break.

---

## The tool: a stocktake, not a vibe check

I didn't write the thing that caught the empty folders myself. It's a slash command, `/skill-stocktake`, that showed up when I pulled in a shared skills collection (its own frontmatter tags it `origin: ECC`). It scans `~/.claude/skills/` and, if you run it from a project root, that project's `.claude/skills/` too, then evaluates every skill against a fixed checklist:

```text
- [ ] Content overlap with other skills checked
- [ ] Overlap with MEMORY.md / CLAUDE.md checked
- [ ] Freshness of technical references verified
- [ ] Usage frequency considered
```

Every skill gets one of five verdicts:

| Verdict | Meaning |
|---------|---------|
| Keep | Useful and current |
| Improve | Worth keeping, specific improvements needed |
| Update | Referenced technology is outdated |
| Retire | Low quality, stale, or cost-asymmetric |
| Merge into [X] | Substantial overlap with another skill; names the target |

Two modes: a **Quick Scan** that only re-evaluates skills changed since the last run (5-10 minutes, driven off a cached `results.json` and file mtimes), and a **Full Stocktake** that walks the entire folder (20-30 minutes, chunked into batches of about 20 skills per subagent call so the context doesn't blow up mid-review). The verdicts are holistic judgment against four dimensions, not a numeric score: actionability, scope fit, uniqueness, and currency. And it's explicit that a bad reason isn't good enough. "Superseded" is rejected. "Superseded by continuous-learning-v2, which covers all the same patterns plus confidence scoring, no unique content remains" is what it expects instead. Vague verdicts don't help you decide anything, so the checklist forces you to write the decision, not just the label.

That's the mechanism. Here's what it actually finds when you point it at real files.

---

## Two real examples from my own folder

**`continuous-learning` vs `continuous-learning-v2`.** Both live in `~/.claude/skills/`, both tagged `origin: ECC`. Here's the full frontmatter of each:

```yaml
# continuous-learning/SKILL.md
name: continuous-learning
description: Automatically extract reusable patterns from Claude Code
  sessions and save them as learned skills for future use.
```

```yaml
# continuous-learning-v2/SKILL.md
name: continuous-learning-v2
description: Instinct-based learning system that observes sessions via
  hooks, creates atomic instincts with confidence scoring, and evolves
  them into skills/commands/agents.
version: 2.0.0
```

Run this through the checklist and the verdict writes itself. Overlap with another skill: yes, direct, the v2 description literally says it evolves into "skills/commands/agents," a superset of what v1 does. Uniqueness: none left in v1. This is a textbook **Merge into continuous-learning-v2**, not because v1 is bad, but because keeping both means every future session has to load two overlapping descriptions to decide which one applies.

**The `google-agents-cli-*` set.** Seven skill names, cloned across two folders, all pointing at an empty `references/` directory with no `SKILL.md`. Actionability: zero, there's no body to act on. This isn't an "Improve" or a "Merge," it's a straight **Retire**, in both locations, because a skill that can't load isn't providing partial value, it's providing none while still occupying a slot in the index.

Neither of these needed a 30-minute Full Stocktake to spot. They needed someone to actually look.

---

## The same checklist applies past SKILL.md

The four-question checklist doesn't care that it was written for skills. It applies to every layer in [the five-file stack](https://hiddedesmet.com/building-a-complete-agent-fleet):

| File type | What "overlap" looks like | What "stale" looks like |
|-----------|---------------------------|--------------------------|
| `AGENTS.md` | Repeats a rule that's already enforced by a linter or `.editorconfig` | References a build command that no longer exists |
| `.instructions.md` | Two files with overlapping `applyTo` globs giving contradictory advice | `applyTo` pattern matches a directory that got renamed |
| `SKILL.md` | Two skills answering the same "when do I fire" question | A skill teaching an SDK version you no longer use |
| `.prompt.md` | A prompt file nobody has invoked in months, copy-pasted into chat instead | Steps reference a tool or command that changed shape |
| `.agent.md` | Two custom agents with near-identical tool restrictions and system prompts | A handoff button pointing at an agent name that got renamed |

If you're only auditing `SKILL.md` files, you're auditing the layer that's easiest to check (there's tooling for it now) and skipping the layer most likely to actively mislead you, which is `AGENTS.md`, because it's always on and nobody wants to touch the one file every request depends on.

---

## The manual version, if you don't have the tool

You don't need a subagent-driven stocktake command to catch the two failure modes above. Both are one-liners.

Find skill folders with no `SKILL.md` at all, across every tool's folder:

```bash
for d in ~/.claude/skills ~/.agents/skills ~/.copilot/skills; do
  [ -d "$d" ] || continue
  find "$d" -mindepth 1 -maxdepth 1 -type d | while read -r skill; do
    [ -f "$skill/SKILL.md" ] || echo "MISSING SKILL.md: $skill"
  done
done
```

Find skill names that exist in more than one folder, which is your shortlist for "check these for actual content overlap first":

```bash
comm -12 <(ls ~/.claude/skills | sort) <(ls ~/.agents/skills | sort)
```

Neither of those tells you *why* a duplicate exists or whether it's safe to delete. That part still needs a human, or an agent, reading both files and applying the four questions: overlap, overlap with the always-on files, currency, usage. But it turns "I should probably review my skills sometime" into a five-second command that hands you a concrete list to start from.

---

## Takeaways

- Skill and agent customization files don't fail loudly when they rot. No build breaks, no lint warning fires. That's exactly why they need a scheduled look, not a "we'll notice when it's a problem."
- A verdict without a reason isn't a verdict. "Superseded" tells you nothing; "superseded by X, no unique content remains" tells you whether to act.
- Duplicate names across tool-specific folders (`~/.claude/skills/`, `~/.agents/skills/`, `~/.copilot/skills/`) are the cheapest thing to check first, before you spend time on deep content review.
- The four-question checklist (overlap with another skill, overlap with always-on files, currency, usage) applies to `AGENTS.md`, `.instructions.md`, and `.agent.md` just as much as it applies to `SKILL.md`. Don't stop the audit at the layer that has a tool for it.
- Run the two one-liners above against your own skill folders before you add skill number 106. There's a decent chance one of them is already empty.
