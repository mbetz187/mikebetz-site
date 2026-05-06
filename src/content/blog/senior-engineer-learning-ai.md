---
title: "What Happens When a Senior Engineer Starts Learning AI"
description: "Twenty years of experience makes some things easier and some things harder. Here's the honest version."
date: 2026-05-06
tags: ["AI", "Career", "Learning"]
---

There's a weird thing that happens when you have 20 years of experience and start learning something new.

You're faster at the fundamentals. Pattern recognition kicks in quickly. You've seen enough architectures, enough hype cycles, enough "revolutionary" tools that didn't pan out. That experience is real and it's valuable.

But it also fights you.

## The Over-Engineering Trap

My biggest obstacle learning AI has not been the concepts. It's been my instinct to design everything before I build anything.

Two decades of enterprise architecture wires you a certain way. You think in systems. You plan for scale. You anticipate failure modes before you write the first line of code. That discipline has served me well across Oracle, SQL Server, SAP Hybris, and a dozen other platforms.

In AI development, it slows you down.

The field moves fast enough that a perfectly designed system built on last quarter's assumptions can already be outdated. The right approach is often to build something small, learn from it, and iterate. That runs directly against the instinct to get the architecture right first.

I've had to consciously override that instinct. It's harder than learning a new framework.

## What Experience Actually Gets You

That said, experienced engineers have real advantages here. They're just different ones than you'd expect.

**Data instincts.** I've spent years working with messy enterprise data. That makes me faster at recognizing when an AI output problem is actually a data quality problem in disguise. Garbage in, garbage out is not a new concept. AI just makes it more visible and more painful.

**Skepticism.** I've lived through the ERP boom, the big data era, and NoSQL mania. I don't get swept up in benchmarks or vendor demos. That keeps me grounded when evaluating AI tooling and helps me ask the right questions about what a model actually does well versus what a demo made it look like.

**System thinking.** LLM applications are still systems. They have inputs, outputs, latency, failure modes, and cost profiles. Building a RAG pipeline or an agentic workflow uses a lot of the same muscles as building a data pipeline. The vocabulary is different but the discipline is the same.

## What Doesn't Transfer

Here's where I'd push back on the "experienced engineers will dominate AI" narrative.

The need for certainty doesn't transfer. SQL is deterministic. Prompt engineering is not. The same prompt can produce different outputs. "Better" is fuzzy. There is no `EXPLAIN PLAN` equivalent. Learning to iterate toward "good enough" without a clean definition of correct is genuinely uncomfortable when you've spent 20 years in a world where the query either returns the right rows or it doesn't.

The mental model of "what's happening inside the system" doesn't transfer cleanly either. I can reason deeply about what a query optimizer does with an index. I cannot reason with the same depth about what a large language model is doing when it hallucinates confidently. That requires a different kind of intellectual humility.

## The Honest Takeaway

If you're a senior technical professional sitting on the sideline waiting for AI to mature, I'd push back on that framing. The tools will keep moving. There's no stable version to wait for.

The best time to build fluency was two years ago. The second best time is now.

Your experience is an asset. Your instincts are mostly right. But some of them will actively slow you down, and the work is figuring out which ones.

That's the part nobody tells you upfront.

More experiments coming. Stay tuned.
