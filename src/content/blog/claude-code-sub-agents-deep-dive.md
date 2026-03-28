---
title: "Claude Code's Hidden Power: Sub-Agents Changed How I Work"
description: "Most developers use Claude Code as a smarter autocomplete. The real unlock is the Agent tool — spawning parallel sub-agents that explore your codebase independently and keep your main context clean."
date: 2026-03-28
tags: ["AI", "Claude Code", "Tools"]
---

First, a quick note. I've been out sick for the past couple of weeks — nothing serious, just the kind of thing that quietly wrecks your schedule. I'm back, the drafts are open, and there's a lot to catch up on.

---

I've written before about using Claude Code to build this site and the recipe manager. In both cases I described it the same way: a pair programmer that eliminates friction. You direct the work, it handles the implementation. That's accurate as far as it goes.

But I've been using it differently lately, and the difference is significant enough to write about.

## What Most People Do

The default Claude Code workflow is linear. You're in a terminal session, you describe a task, it reads some files, writes some code, and you review it. That works well for focused, bounded work — adding a feature to a file you're already looking at, fixing a bug you've already localized, asking a question about code you understand.

It breaks down when the scope is wider. Larger codebases. Multi-file refactors. "How does X actually work in this project?" questions where the answer might live in five different places. When the task requires exploration before implementation, linear mode gets slow and the context window fills up fast.

## The Context Window Problem

Here's the thing most people don't think about explicitly: every file Claude reads, every piece of code it writes, every exchange in a session — it all accumulates in the context window. That window is large, but it's finite. And a bloated context isn't just a technical limit; it actively degrades quality. The more noise in the window, the less precisely the model can attend to what actually matters.

I noticed this first on a task that required understanding how data flowed across several interconnected files before touching any of them. By the time we'd done the exploration, the context was cluttered with intermediate dead ends and the actual implementation felt fuzzier than it should have.

That's when I started using the Agent tool deliberately.

## Sub-Agents: What They Actually Are

Claude Code has an Agent tool that can spawn specialized sub-agents to handle discrete tasks. These sub-agents run with their own isolated context windows. When they're done, they return a focused result — not their full session history — back to the main conversation.

Think of it the way you'd think about parallel database queries. Instead of one long sequential scan across your whole schema, you fire off targeted queries that each return exactly the data you need. The sub-agents do the exploration; you get the summary.

There are a few agent types that matter:

- **Explore** — fast, read-only, built for codebase searches. Give it a specific question; it comes back with an answer.
- **Plan** — designed for designing implementation approaches before any code is touched. Explores first, then produces a structured plan.
- **General-purpose** — runs the full tool suite, good for tasks that require both research and execution.

The power move is running multiple Explore agents **in parallel** when you need to understand several parts of a codebase at once.

## What This Looks Like in Practice

When I was adding structured tag filtering to this blog, I needed to understand three separate things before touching anything:

1. How the existing content collections schema was structured
2. How Astro's routing handled dynamic paths
3. Where tag references already existed across the templates

Old approach: read each area sequentially in one long session. New approach: launch three Explore agents simultaneously — one per question — each operating in its own clean context. They run in parallel, come back with focused answers, and I have everything I need without a context window full of tangential file reads.

The actual implementation session starts clean. The agents did the archaeology; I write the code.

## The Other Benefit: Protecting Your Main Session

Beyond speed, there's a quality benefit that's harder to quantify but real.

When you send an Explore agent into a dense area of a codebase, it might read dozens of files chasing down a dependency chain. If all of that happened in your main session, that content would sit in the context for the rest of the conversation — potentially confusing things, definitely consuming space. With sub-agents, the deep-dive happens in a throwaway context. You get the answer, not the mess.

I started thinking of it as the difference between having a conversation with someone who has total recall of every tangent you've ever gone down together, versus someone who went away, did focused research, and came back with exactly what you needed to know.

## Where It Actually Helps

To be specific about where sub-agents earn their keep and where they don't:

**Worth it:**
- Unfamiliar codebases or areas of the project you haven't touched in a while
- Tasks that require understanding multiple files before you know which one to change
- Anything where you'd naturally say "I need to look around a bit before I start"
- Large refactors where you want a plan before code gets touched

**Not worth it:**
- Small, localized changes where you already know the file
- Simple questions about code you're currently looking at
- Quick fixes where the diagnosis and solution are obvious

The overhead of spinning up agents isn't zero. For targeted, bounded work, just work directly.

## The Honest Take

This isn't a feature hidden behind a setting or a flag you have to enable. It's just the Agent tool, available in every Claude Code session. Most people don't use it intentionally because the linear workflow feels natural — it's how chat interfaces have always worked.

But Claude Code isn't a chat interface. The architecture is genuinely different: tool-using, composable, designed around the idea that a model with access to sub-agents is fundamentally more capable than one without them.

Once you start thinking in terms of "what needs exploration, and can I parallelize it?" the tool starts working differently. Not dramatically faster in every session, but meaningfully better on the tasks where scope and context are the real bottleneck.

That's most of the interesting tasks.
