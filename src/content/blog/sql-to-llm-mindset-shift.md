---
title: "From SQL to LLMs: The Mindset Shift No One Talks About"
description: "What two decades of database architecture taught me about thinking in probabilities instead of certainties."
date: 2026-03-01
tags: ["AI", "Databases", "Architecture"]
---

When you write a SQL query, you know exactly what you're going to get. Given the same data and the same query, the result is deterministic. That's the game I've played for 20+ years across Oracle, SQL Server, and T-SQL.

LLMs do not work that way. And that's a harder mental shift than it sounds.

## The Certainty Trap

Experienced engineers have deep intuition for deterministic systems. We think in schemas, constraints, and foreign keys. We design for predictability. We distrust systems we can't fully reason about.

LLMs are fundamentally probabilistic. The same prompt can produce different outputs. "Better" is fuzzy. Evaluation is hard. There's no `EXPLAIN PLAN` to tell you why the model chose those words.

## What Actually Transfers

The good news: enterprise architecture thinking transfers more than you'd expect.

**Data quality instincts** — Garbage in, garbage out applies to both SQL queries and LLM prompts. My instinct to validate inputs and structure data cleanly is directly useful in prompt engineering and RAG pipelines.

**System design** — LLM applications are still systems. They have inputs, outputs, failure modes, and scaling constraints. Designing robust AI pipelines uses the same muscles as designing data pipelines.

**Skepticism** — Vendor hype cycles are nothing new. I've lived through the ERP boom, the big data era, and NoSQL mania. That skepticism keeps me grounded when evaluating AI tooling.

## What Doesn't Transfer

**The need for precision.** In SQL, a query is right or wrong. In prompt engineering, you're tuning toward "better" with fuzzy metrics. Learning to work in that space without the safety net of correctness is uncomfortable.

**Mental model of "what the system is doing."** I can reason deeply about what a query optimizer does. I cannot reason with the same depth about what happens inside a 70B parameter model. That's a new kind of intellectual humility.

## The Takeaway

If you have a strong technical background and you're approaching AI, don't assume your instincts are wrong — many of them are right. But be intentional about identifying where the rules have changed.

The engineers who will do this best are the ones who bring disciplined thinking *and* can tolerate ambiguity. Both matter.

More experiments coming. Stay tuned.
