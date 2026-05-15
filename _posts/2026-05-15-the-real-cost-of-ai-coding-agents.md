---
layout: post
title: "The real cost of AI coding agents: what your team actually spends"
date: 2026-05-15
categories: [AI]
tags: [ai, copilot, claude-code, openai, anthropic, github-copilot, pricing, developer-tools, agents]
author: hidde
description: "A quick chat costs about $0.0015. A heavy agent session costs $5. GitHub just admitted flat-rate pricing can't survive this gap. Here's what AI coding agents actually cost at team scale, with real numbers from official sources."
image: /images/freelunch.png
featured: true
toc: true
---

A quick chat question using GPT-5 mini costs roughly $0.0015. A multi-file refactor using GPT-5.5 in agent mode costs around $5.50. Same subscription, same user, more than 3,600x cost difference.

GitHub said it plainly in their April 27 announcement: "The current premium request model is no longer sustainable." They are switching every Copilot plan to usage-based billing on June 1, 2026. That means your team's AI bill is about to become variable, and most engineering managers have no idea what the actual numbers look like.

Here is what AI coding agents cost right now, across every major provider, with official pricing you can verify yourself.

---

## GitHub Copilot: the new pricing model

Base plan prices stay the same. What changes is how usage is metered.

| Plan | Monthly price | Included usage from June 1 | Notes |
|------|---------------|----------------------------|-------|
| **Free** | $0 | Limited AI credits + 2,000 completions/month | Auto model selection |
| **Pro** | $10/user | 1,500 credits ($15 total allowance) | $10 base + $5 flex |
| **Pro+** | $39/user | 7,000 credits ($70 total allowance) | $39 base + $31 flex |
| **Max** | $100/user | 20,000 credits ($200 total allowance) | $100 base + $100 flex |
| **Business** | $19/user | 1,900 credits/user, pooled | Promo Jun–Aug: 3,000 credits ($30) |
| **Enterprise** | $39/user | 3,900 credits/user, pooled | Promo Jun–Aug: 7,000 credits ($70) |

Starting June 1, every chat message, agent session, and code review consumes tokens. Those tokens convert to **GitHub AI Credits** at a fixed rate: 1 credit = $0.01 USD. Code completions and Next Edit Suggestions remain unlimited and free on all paid plans. Copilot code review is the exception to watch: GitHub says it will consume both AI Credits and GitHub Actions minutes on GitHub-hosted runners.

The pooling model matters. A 50-person Business org gets a shared pool of 95,000 credits ($950 worth). Power users draw more, light users offset them. If you blow through the pool, you either pay overage or get cut off, depending on your admin's budget configuration.

**Promotional period (June–August 2026):** existing Business seats get 3,000 credits/user/month. Existing Enterprise seats get 7,000 credits/user/month. After that, the included org allowance drops to the standard 1,900 and 3,900 credits/user amounts.

**The base vs flex split is the part to read twice.** GitHub split each individual plan's included usage into two parts: **base credits** (matched 1:1 with your subscription price) and a **flex allotment** on top. On Pro you get $10 base + $5 flex, on Pro+ it is $39 base + $31 flex, on Max it is $100 base + $100 flex. GitHub has been explicit that the flex portion is variable: it "is designed to adapt as the economics of AI evolve, including model pricing, new models, and improvements in efficiency." Read that as: only the base is contractually guaranteed. If model costs go up or efficiency does not improve as fast as GitHub hopes, the flex allotment can shrink. So when you size a plan, size it on the base credits and treat flex as a bonus that may not last.

**My take on which plan to actually buy.** Pro at $10 is a trap if you use agent mode at all. 1,500 credits is roughly two or three serious agent sessions per month before you hit overage, and only 1,000 of those credits are actually guaranteed. If you are a solo developer who lives in agent mode, Pro+ at $39 is the realistic floor and Max at $100 is honest pricing for what heavy use actually costs. For teams, do not let finance pick Business on price alone. Run the math on your top three power users first. If even two of them burn through 5,000 credits a month, the pooled allowance disappears fast and you are funding their habit out of everyone else's budget.

Source: [GitHub blog](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/), [GitHub individual plans update](https://github.blog/news-insights/company-news/github-copilot-individual-plans-introducing-flex-allotments-in-pro-and-pro-and-a-new-max-plan/), [GitHub docs](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises)

---

## Per-token rates on GitHub Copilot (effective June 1)

One important detail: GitHub Copilot is not one model. It has been a multi-vendor picker for a while. OpenAI, Anthropic, Google, and xAI all sit behind the same UI. What changes on June 1 is the billing. Under flat-rate pricing, GitHub absorbed the per-vendor cost difference. Under usage-based billing, you see it. A token on Claude Opus 4.7 has always been more expensive for GitHub to serve than a token on GPT-5 mini. Now that gap shows up on your invoice. GitHub's billing docs also reference a separate **Fine-tuned (GitHub)** category, but those entries are not standard picker options today, so expect to only see them if your org has them enabled. The practical point is simpler: your cost profile changes depending on which model a developer picks, and which models your admin has turned on.

All prices per 1 million tokens:

| Model | Input | Cached input | Output |
|-------|-------|--------------|--------|
| GPT-5 mini | $0.25 | $0.025 | $2.00 |
| GPT-4.1 | $2.00 | $0.50 | $8.00 |
| GPT-5.4 | $2.50 | $0.25 | $15.00 |
| GPT-5.5 | $5.00 | $0.50 | $30.00 |
| Claude Haiku 4.5 | $1.00 | $0.10 | $5.00 |
| Claude Sonnet 4.6 | $3.00 | $0.30 | $15.00 |
| Claude Opus 4.7 | $5.00 | $0.50 | $25.00 |
| Gemini 2.5 Pro | $1.25 | $0.125 | $10.00 |
| Gemini 3 Flash (Preview) | $0.50 | $0.05 | $3.00 |
| Gemini 3.1 Pro (Preview) | $2.00 | $0.20 | $12.00 |
| Grok Code Fast 1 (admin-gated) | $0.20 | $0.02 | $1.50 |

Watch the output-to-input ratio. Claude Opus 4.7 charges 5x more for output tokens than input tokens. Agent sessions that generate a lot of code (writes, refactors, multi-file edits) hit that output multiplier hard.

**What I actually recommend defaulting to.** For day-to-day chat and small edits, GPT-5 mini or Claude Haiku 4.5 are the right answer. They cost roughly a tenth of the flagship models and they are good enough for explaining errors, writing tests, and one-file changes. Use Claude Sonnet 4.6 as your default agent model. It is the best price-to-quality point in the table for multi-file work. Save Opus 4.7 and GPT-5.5 for things that genuinely need them: hard debugging, gnarly refactors, or architecture work where a wrong answer costs you an hour. If your team's instinct is to always pick the most expensive model because it sounds safer, you are paying a 5x to 20x premium for a difference most tasks will not notice.

Source: [GitHub Copilot models and pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing), [GitHub Copilot plan comparison](https://docs.github.com/en/copilot/get-started/plans)

---

## Anthropic: Claude Code and API pricing

Anthropic runs two billing models for developers. You either use the consumer subscription (Claude Pro/Max, which includes Claude Code), or you call the API directly.

### Consumer plans (includes Claude Code)

| Plan | Price | Usage |
|------|-------|-------|
| Free | $0 | Limited |
| Pro | $20/month, or $17/month billed annually | Standard usage, Claude Code included |
| Max 5x | $100/month | 5× Pro usage |
| Max 20x | $200/month | 20× Pro usage |

### API pricing (per 1M tokens)

| Model | Input | Output | Cache hit (90% off) |
|-------|-------|--------|---------------------|
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 |
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 |

Anthropic also publishes a worked example for their Managed Agents product: a 1-hour coding session on Opus 4.7 consuming 50,000 input tokens and 15,000 output tokens costs **$0.71** (tokens + $0.08 session runtime).

With prompt caching active (which agent sessions almost always trigger), that drops to **$0.53**.

**Subscription or API: how to choose.** If you are a solo developer using Claude Code most days, the $20 Pro subscription pays for itself by lunchtime on the first heavy session. If you are building a product that calls Claude programmatically, the API is the only option and prompt caching is non-negotiable. The trap to avoid is paying for both. I have seen teams expense individual Max subscriptions while also running an API budget for the same workflows. Pick a lane per use case.

Source: [Anthropic API pricing](https://platform.claude.com/docs/en/docs/about-claude/pricing)

---

## OpenAI API pricing

| Model | Input | Cached | Output |
|-------|-------|--------|--------|
| GPT-5.4 mini | $0.75 | $0.075 | $4.50 |
| GPT-5.4 | $2.50 | $0.25 | $15.00 |
| GPT-5.5 | $5.00 | $0.50 | $30.00 |

GPT-5.5 output at $30/MTok is among the most expensive rates on current-generation flagship models. (Anthropic's older Claude Opus 4.1 is still listed at $75/MTok output, but it is not where most coding agents will route work today.) A heavy agent session that generates 100k output tokens on GPT-5.5 costs $3.00 in output alone.

Source: [OpenAI API pricing](https://openai.com/api/pricing/)

---

## Google Gemini API pricing

Gemini is worth adding to the comparison because it sits in two places: directly through Google's Gemini API, and indirectly inside GitHub Copilot's model picker for supported plans.

| Model | Input | Cached | Output |
|-------|-------|--------|--------|
| Gemini 3.1 Pro Preview | $2.00 | $0.20 | $12.00 |
| Gemini 3 Flash Preview | $0.50 | $0.05 | $3.00 |
| Gemini 2.5 Pro | $1.25 | $0.125 | $10.00 |
| Gemini 2.5 Flash | $0.30 | $0.03 | $2.50 |
| Gemini 2.5 Flash-Lite | $0.10 | $0.01 | $0.40 |

Those numbers make Gemini interesting at the low and middle end. Gemini 2.5 Flash-Lite is extremely cheap for high-volume simple work. Gemini 2.5 Pro lines up exactly with Copilot's Gemini 2.5 Pro pricing. Gemini 3.1 Pro is closer to a frontier-model slot, but still cheaper than GPT-5.5 or Claude Opus 4.7 on output tokens.

One caveat: Google's pricing page also includes separate charges for tools such as grounding with Google Search, Google Maps, URL context, file search, and code execution. If your agent uses those tools heavily, token pricing is not the whole bill.

Source: [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing)

---

## What a typical session actually costs

Here is what real usage patterns look like in dollar terms, calculated from the published per-token rates above:

| Scenario | Model | ~Tokens (in / out) | Cost |
|----------|-------|---------------------|------|
| Quick chat question | GPT-5 mini | 2k / 500 | $0.0015 |
| Code review of a PR | Claude Sonnet 4.6 | 50k / 10k | $0.30 |
| 30-min agent session | Claude Sonnet 4.6 | 100k / 30k | $0.75 |
| 1-hour deep agent session | Claude Opus 4.7 | 200k / 50k | $2.25 |
| Multi-file refactor | GPT-5.5 | 500k / 100k | $5.50 |
| All-day heavy agent user | Mixed models | ~2M / 500k | $15–25 estimate |

The gap between a chat question and a heavy agent day is roughly **10,000x to 17,000x** against the quick-chat example above. That is why flat-rate pricing broke.

---

## What this means for a 10-person team

Let's run the math on a Copilot Business team of 10 engineers:

**Base cost:** 10 × $19 = $190/month

**Included credits:** 10 × 1,900 = 19,000 AI credits ($190 worth)

**Illustrative consumption scenarios:**

These are estimates, not GitHub-published usage averages. They assume 20 workdays per month and a blended mix of chat, code review, and agent sessions.

| Team type | Daily usage pattern | Monthly credit burn | Overage |
|-----------|--------------------|--------------------|---------|
| Light (mostly completions) | A few chats, rare agent use | ~8,000 credits | None |
| Moderate (regular agent use) | About $12.50/day across the team | ~25,000 credits | ~$60 |
| Heavy (agents as default) | About $25/day across the team | ~50,000 credits | ~$310 |

A heavy team's real cost: $190 base + $310 overage = **$500/month** for 10 people, or $50/person.

That is still cheap compared to the time saved. But it is 2.6× what the sticker price says. Engineering managers who budget $19/seat/month are going to get a surprise in July.

**What I would actually budget for.** Plan for $40 to $60 per seat per month for any team that takes agent mode seriously. That gives you headroom without conversations about who used too many credits this week. The wrong move is to under-budget, then introduce restrictive per-user caps the first time the bill spikes, then watch your engineers stop using the tool because they are scared of going over. AI coding only pays back when people use it freely. A $30/month overage is rounding error against an engineer's salary; a culture where people avoid the tool is not.

---

## The cost optimization levers

Four things you can control:

**1. Model selection matters enormously.** The same task on GPT-5 mini ($0.25 input) versus GPT-5.5 ($5.00 input) is a 20× difference. Haiku 4.5 versus Opus 4.7 is 5× on input, 5× on output. If your agent can do the job on Sonnet, do not run it on Opus.

**2. Prompt caching is free money.** Cached input tokens cost 90% less on Anthropic and 90% less on GitHub's rates too. Agent sessions that reuse context (same repo, same system prompt, iterating on the same files) benefit automatically. This is already active in most coding agents.

**3. Budget controls exist.** GitHub lets admins set budgets at enterprise, org, cost-center, and individual user levels. Set them before June 1. A $0 user-level budget means no Copilot access at all, so be deliberate.

**4. Model governance is a platform decision.** In GitHub Copilot Business and Enterprise, admins can manage which Copilot features and models are available to an organization. GitHub's docs call out a separate **Models** policy area for models beyond the basic ones, because some may incur additional costs. Use that. If your tooling supports Auto model selection and you trust it, standardize on Auto for routine work. If not, restrict the expensive models to the people and workflows that actually need them. You do not want Claude Opus or GPT-5.5 burning credits to explain a `git fetch` error.

---

## The bigger picture

The pricing signals are clear. Every major provider is converging on the same model: cheap base subscriptions, variable costs based on actual compute.

- GitHub is moving from request-based to token-based billing
- GitHub Copilot has been multi-vendor for a while; usage-based billing just makes the per-vendor cost visible to the buyer, with some models (xAI, fine-tuned previews) gated behind admin policy
- Anthropic offers fixed subscriptions (Pro and Max) alongside per-token API access
- OpenAI has always been pay-per-token on the API side
- Google Gemini is competitive on cheaper and mid-tier agent workloads, especially Flash and Flash-Lite

The teams that will pay the least per unit of value are the ones that pick the right model for each task, cache aggressively, and stop using frontier models for tasks that a $0.25/MTok model handles fine.

The teams that will get a surprise bill are the ones running Claude Opus 4.7 on every autocomplete because it was the default, with no budget guardrails set.

**The honest part nobody is saying out loud.** Expect the rest of the market to follow GitHub here. Cursor at $20/month, Claude Code on a $20 Pro subscription, Codeium, Windsurf, and every other flat-rate AI coding tool is doing the same math GitHub just published, and the answer is the same: heavy agent users cost more to serve than the subscription brings in. Right now those vendors are absorbing the gap, funded by either VC money or by the light users subsidizing the heavy ones. That ends. Either prices go up, included usage goes down, or the whole industry shifts to metered billing within the next 12 to 18 months. If you are picking a tool today purely because it is cheaper than Copilot, you are timing a market that is about to reprice. Pick on quality and fit instead, and budget for the real per-token cost, because that is where everyone is heading.

My honest advice for the next 30 days: pick a default model that is not Opus or GPT-5.5, turn on Auto where it is supported, set a per-user budget so nobody gets cut off cold, and look at the usage dashboard once a week for the first two months. After that you will know your real number, and the conversation with finance gets a lot easier.
