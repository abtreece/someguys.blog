---
layout: post
title: Twenty years in infrastructure, and the work doesn't look the same anymore
date: 2026-05-09
tags: ["ai", "career"]
---

For the past couple of months I've been asking myself whether twenty years of experience still counts for anything, and how I'm supposed to describe the work on my LinkedIn profile if AI is doing all the work. The engineers who don't use AI, or refuse to, aren't helping. "Ask your robot friend." "Is this AI slop?" "Looks vibe coded." Each one is a way of saying the work doesn't count because of how it got done. The shade is loud. The doubt is mine. So this post is me writing my way to an answer.

<!--more-->

The 2.0 release of Claude Code landed at the end of September 2025, and at some point, between Halloween and Thanksgiving, AI went from "this is interesting" to "this is a step change". I spent the break between Christmas and New Year's really digging into Claude Code. A few weeks later and I'm delegating around half my daily work to Claude.

## From writing code to directing agents

For most of my career, the loop has been the same. A problem would surface or a change request would land, I'd think through the architecture, weigh the tradeoffs, write the design doc, write the code, push the PRs, review the code, coordinate the rollout, and watch the dashboards. Then the next requirement would land and I'd do it again.

That loop has collapsed.

I still own the architecture and set the constraints. I still bring the domain context the model does not have: why our deployment patterns look the way they do, why this database needs that failover strategy, why a specific migration approach is the only one that won't cause issues in production. I direct agents through multi-step work, verify what came back, iterate on it, and ship. This isn't vibe coding. It's a Principal Engineer with twenty years of scar tissue using AI to move a lot faster.

## The industry is still figuring out what to call it

I'm not the only one trying to put words to this.

Nicholas Zakas, who created ESLint, [argues](https://humanwhocodes.com/blog/2026/01/coder-orchestrator-future-software-engineering/) that the software engineering job of the future won't involve writing code at all. It will involve orchestrating AI agents that write the code.

[Builder.io](https://www.builder.io/blog/ai-software-engineer) put it more bluntly: the 10x engineer of 2026 is somebody who has learned to effectively manage ten agents.

In February, Boris Cherny predicted on [Lenny's Podcast](https://www.lennysnewsletter.com/p/head-of-claude-code-what-happens) that "by the end of the year everyone is going to be a product manager, and everyone codes. The title software engineer is going to start to go away." Also in February, Boris was asked on X why Anthropic still has 100+ open engineering roles if Claude Code is writing all the code. [His answer](https://x.com/bcherny/status/2022762422302576970):

> Someone has to prompt the Claudes, talk to customers, coordinate with other teams, decide what to build next. Engineering is changing and great engineers are more important than ever.

Both things are true, and they're not in tension. The title will change because the work has changed. The work that's left is the work that mattered most all along.

## The delegation delta is where senior engineers live

[Anthropic's 2026 Agentic Coding Trends Report](https://resources.anthropic.com/2026-agentic-coding-trends-report) has the stat that sticks with me: engineers use AI in about 60% of their work but can only fully delegate 0 to 20% of tasks.

That gap is the whole story.

Sixty percent of the work touches AI. Less than twenty percent of it can be fully handed off. Everything in between is where experienced engineers earn their seat.

That space between is where the work lives now: judging the output, reviewing it carefully, integrating it into a system that already has opinions, and escalating when something doesn't add up. You can look at an agent's output and say "yes, but not like this," and know why. You understand that a migration plan can be technically correct but operationally catastrophic. You know to push back on a plausible-looking architecture because you've seen it fail before, in production, at 3am, on a Saturday.

Judgment hasn't been automated. A model can produce code that compiles and tests that pass, but it can't tell me what's worth building, how it should behave in production, or why a migration belongs at 2am and not 2pm. That part still takes twenty years of running services at scale.

What's been automated is the mechanical translation of decisions into code. What changes is what an experienced engineer can ship in a week.

## What's actually working for me

A few practices have become non-negotiable.

**Context is leverage.** Agents are only as useful as the environment you give them, which means your architecture docs, conventions, and the reasoning behind past decisions all need to be discoverable. The engineers who do well here are the ones who have already invested in writing down their thinking, not just their code.

**Spec before agent.** Spec means what it has always meant: an RFC, a design doc, an ADR. These are the artifacts I'd write before any code got cut whether or not an agent was involved, reviewed by other engineers the same way they always have been. What's added is a plan that my robot friend and I both agree on before any work starts, with acceptance criteria, constraints, what shouldn't be touched, and what must stay backward-compatible all spelled out. The artifacts are the spec; the agreed plan is the contract. The more specific I am up front, the less rework I'm doing on the back end.

**Tests are the contract.** TDD got more important, not less. When an agent is writing the implementation, the tests and the CI pipeline are how I know it did what I asked, not what it thought I asked. I write the test first, or I have the agent write the test against the spec and I review it before any production code is touched. Then the CI gates do their job: lint, unit, integration, security scans, the works. An agent's output isn't done because it looks done. It's done when the pipeline says it's done.

**Review discipline.** AI that's almost right is the most dangerous AI. Code that looks correct and fails in production is a new class of defect. Reviewing it takes an instinct that assumes nothing and verifies the assertions.

**Parallel streams.** When one agent is running, I'm specifying the next. The workflow used to be serial: think, write, test, review. Now it's a pipeline. Keeping it full without overwhelming my own review capacity is its own skill, and most engineers haven't needed to develop it before.

## A note for the senior engineers quietly wondering

If you're fifteen, twenty, twenty-five years into your career and you're wondering whether the experience still counts: it counts more than ever.

The floor of what any one engineer can ship just went way up. A subscription and a laptop will get you something that runs. The ceiling went up further than that, but only if you've done the unglamorous work of building the judgment that separates "runs" from "runs reliably at peak load during market hours".

Your value didn't decrease. It moved up the stack.

Our job description has changed; HR just doesn't know it. The best move right now is to describe the shift on your own terms. Write the post. Update the headline. Name the work.

Here's mine: I'm a Principal Infrastructure Engineer who orchestrates AI agents to deliver production-grade payment infrastructure at a velocity that was not possible eighteen months ago. The infrastructure problems didn't get easier. The tooling got a lot better. And twenty years of experience is what makes that tooling produce output worth shipping.

I'm convinced, this is the job now.
