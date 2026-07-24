---
title: "I Pointed Three Agents at the Same Codebase. Here's What Broke."
description: "An experiment in running multiple AI agents in parallel on real backlog items from the recipe manager — what worked, what collided, and the one boundary that mattered more than any prompt."
date: 2026-07-24
tags: ["AI", "Agents", "Build Log"]
---

Back in March I wrote about the recipe manager and listed three things I'd punt on for later: JSON-file storage that wouldn't scale, no image caching, and no meal plan history. Backlog items, all independent-looking on paper. So this month I tried something: instead of working through them one at a time in a single session, I dispatched three agents at once, one per item, and let them run in parallel.

The idea wasn't exotic. Explore-then-build pipelines are old news at this point. What I hadn't actually tested was whether "independent" backlog items are as independent as they look once real agents start touching real files.

## The Setup

Three tasks, three agents, each in its own git worktree so they couldn't step on each other's working directory:

- **Agent A** — migrate `lib/db.ts` from JSON file I/O to SQLite
- **Agent B** — add local image caching for scraped recipe photos
- **Agent C** — add meal plan history so old plans don't get overwritten

Each got a Plan pass first, then a build pass, then I ran a review pass across the diff before merging. On paper this is the pipeline from my sub-agents post, just fanned out three ways instead of run sequentially in one session.

## What Went Right

Agent B (image caching) was clean start to finish. It's genuinely isolated — new field on the recipe object, a fetch-and-store routine, done. No shared surface with the other two. This is the case parallel agents are actually good at: scope that doesn't touch anything the other agents care about.

Agent C (meal plan history) was almost as clean, with one hiccup: it assumed `writeDb()` was safe to call concurrently with whatever Agent A was doing to the same file. It wasn't wrong given what it could see — it just couldn't see that the ground under `lib/db.ts` was about to move.

## What Collided

Agent A's SQLite migration was, correctly, the deepest change — and it touched the one file both other agents depended on: `lib/db.ts`. All three agents had read that file at roughly the same starting point. Two of them wrote code against an interface that had since changed underneath them.

The merge wasn't catastrophic, but it wasn't free either. I spent longer reconciling the three branches than I would have spent just running the SQLite migration first, then the other two afterward, sequentially, against the finished interface.

The more interesting failure was subtler: Agent A's own verification step ran its test data against a fixture file that didn't match the actual shape of my production JSON — a couple of recipes with legacy fields from before I added ingredient normalization. The migration script "passed" against clean fixtures and would have silently dropped those fields on real data. The review pass caught it. A confident agent report didn't.

## The Actual Lesson

"These tasks look independent" is not the same as "these tasks don't share a dependency." The real boundary for parallelizing agents isn't the task description, it's the file graph underneath it. Anything that touches a shared interface — a shared data layer, a shared schema, a shared config — needs to either go first, sequentially, or get explicitly walled off with a contract the other agents build against instead of the live file.

This is the same shape of problem I wrote about in the governance post, just at a much smaller scale: the failure mode isn't "the agent did something wrong," it's "the agent did something correct in a context that changed without its knowledge." Scope boundaries and a verification step that doesn't trust the agent's own self-report are the same fix whether you're running three agents on a hobby project or thirty in production.

## What I'd Do Differently Next Time

Sequence anything that touches a shared file first, and only fan out the genuinely leaf-node work after that interface is stable. And don't skip the independent review pass — the one bug that would have actually cost me data was invisible to the agent that introduced it and obvious to the one checking it afterward.

Two out of three parallel agents is a fine batting average for a first attempt. The real result of the experiment wasn't the SQLite migration — it was learning where the line is between "looks parallel" and "is parallel." That's worth more than the feature.
