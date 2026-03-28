---
title: "I Built a Recipe App That Plans Meals and Writes My Grocery List"
description: "A full-stack Next.js app that imports recipes from any website, generates weekly meal plans, and consolidates ingredients into a smart shopping list."
date: 2026-03-13
tags: ["AI", "Build Log", "Next.js", "TypeScript"]
---

Every week it was the same thing. Thirty open browser tabs of recipes I might make. A grocery list written on the back of an envelope. Standing in the kitchen at 6pm trying to remember what I was supposed to cook. There are apps that solve this. I tried a few. None of them worked the way I think.

So I built one.

The result is a local-first recipe manager that imports recipes from any cooking website, generates randomized meal plans for the week, and automatically consolidates all the ingredients into a single grocery list. Here's how it works and what I learned building it.

## The App in 30 Seconds

Three core functions:

- **Import** a recipe by pasting a URL — AllRecipes, Food Network, NYT Cooking, Epicurious, or anywhere else
- **Generate** a meal plan: pick the number of days, get a randomized selection from your recipe library
- **Shop**: the app aggregates all ingredients across every recipe in the plan and produces a single deduplicated grocery list with checkboxes

Stack: **Next.js 14** with the App Router, **TypeScript** throughout, **Tailwind CSS** for styling. No database — the whole thing runs off a local JSON file. No auth, no server to maintain. It runs on my machine and does exactly what I need.

## The Recipe Scraper

The most technically interesting piece is the importer. Paste a URL, the app fetches the page and extracts structured recipe data: title, ingredients, instructions, servings, prep/cook times.

The challenge is that recipe sites all implement their markup differently, even when they're technically following the same schema.org standard. I used a dual-mode approach.

**Primary:** The `recipe-scraper` npm library handles 100+ sites out of the box. It knows the quirks of specific domains and handles parsing reliably across the sites people actually use.

**Fallback:** When the library fails or hits an unsupported site, a custom parser using `cheerio` extracts JSON-LD structured data directly from the page's `<script>` tags. Most modern recipe sites include schema.org `Recipe` markup for SEO — we just read it.

One practical detail: the scraper sends a browser-style User-Agent header. Without it, some sites block the request entirely. It's a small thing that matters.

Reliability over purity. Two modes means more recipes import successfully.

## The Ingredient Normalization Problem

Generating a grocery list sounds simple until you actually do it. Consider a meal plan with three recipes:

- Recipe A calls for "1 tablespoon butter"
- Recipe B calls for "2 tbs unsalted butter, softened"
- Recipe C calls for "3 tablespoons of butter, melted"

Without normalization, you get three separate line items. With it, you get "6 tbsp butter."

The normalization logic in `lib/ingredients.ts` handles this in three passes:

1. **Unit normalization** — A `UNIT_MAP` translates every known variation to a canonical form. "tablespoon", "tablespoons", "tbs", "tbsp" all become "tbsp". Same for cups, ounces, grams, and a dozen others.

2. **Name cleaning** — A `FILLERS` list strips descriptive words that don't affect what you buy: "chopped", "diced", "fresh", "softened", "melted", "large", "small". A regex removes parenthetical notes entirely.

3. **Aggregation** — Each normalized ingredient is keyed as `ingredientName::unit`. Entries with the same key get their quantities summed. The final list is sorted alphabetically with unchecked items first.

The result is a grocery list that reads like a human wrote it, not a data dump of every ingredient from every recipe.

## Meal Planning

The meal plan generator takes a number (1–14 days) and randomly assigns a recipe to each day. I used a **Fisher-Yates shuffle** to select without repetition when the recipe library is large enough. If you only have five recipes but want a seven-day plan, it wraps around using modulo rather than failing.

One architectural decision I liked: the meal plan stores `recipe_ids`, not full recipe objects. The UI looks up recipes by ID at render time. Smaller storage, single source of truth, no synchronization bugs if a recipe gets edited after a plan is created.

The shopping list is generated at plan-creation time and stored with the plan. Toggle checkboxes as you shop — items re-sort with purchased items moving to the bottom.

## The API Layer

Next.js App Router handles both frontend and backend in the same project. Every API route lives in `app/api/` as a `route.ts` file — no separate Express server, no additional process to run.

The full API surface:

| Method | Route | Purpose |
|--------|-------|---------|
| `GET` | `/api/recipes` | Fetch all recipes |
| `POST` | `/api/recipes` | Create a new recipe |
| `GET` | `/api/recipes/[id]` | Fetch a single recipe |
| `PUT` | `/api/recipes/[id]` | Update a recipe |
| `POST` | `/api/scrape` | Submit a URL, get back structured recipe data |
| `GET` | `/api/meal-plan` | Fetch the latest meal plan |
| `POST` | `/api/meal-plan` | Generate a new meal plan |
| `GET` | `/api/shopping-list` | Fetch shopping list from the active plan |
| `PATCH` | `/api/shopping-list` | Toggle an item's checked status |

A few design decisions worth noting:

**Scraping is a separate endpoint.** `POST /api/scrape` returns raw parsed recipe data without saving anything. The client then populates the recipe form, the user can review and edit, and a separate `POST /api/recipes` saves it. That separation means the scraper doesn't have side effects — it's a pure transformation.

**Meal plan generation is idempotent by intent.** Each `POST /api/meal-plan` creates a new plan and replaces the previous one. The app only ever works with the latest plan. No history, no versioning — that simplicity was a deliberate call.

**`PATCH` for shopping list toggles.** Rather than `PUT` (full replace) or a custom verb, `PATCH` is the right semantic for a partial update. The request body is just `{ id, checked }` — the server updates that item, re-sorts the list, and persists it. The client uses **optimistic updates**: the checkbox flips immediately in the UI, and the API call happens in the background. If it fails, it reverts. For a local tool this almost never matters, but it's the right pattern.

All routes read and write through `lib/db.ts`, which wraps synchronous JSON file I/O behind `readDb()` and `writeDb()` functions. Every route gets a consistent interface to the data layer regardless of what's underneath it — swap the file for SQLite later and only `db.ts` changes.

## What I'd Do Differently

**JSON file storage** works fine for a personal tool. I have about 40 recipes and it's instant. But this approach tops out around a few hundred recipes before you'd want a real database. SQLite would be the right next step — same zero-infrastructure approach, but proper querying and no whole-file reads/writes on every operation.

**The scraper has gaps.** Some recipe sites use unusual schema implementations or load content dynamically via JavaScript. Those fail silently. I handle it gracefully, but I know some URLs won't import automatically and require manual entry. That's an acceptable tradeoff for a personal tool.

**No image handling.** The app stores image URLs but doesn't cache them locally. If a recipe site goes down, the images disappear. Not a real problem in practice, but worth noting.

## The Honest Take

This isn't a sophisticated app. It's a focused tool that solves a specific problem I actually had. The technical decisions were driven by simplicity: no database, no auth, no deployment, no maintenance. It lives on my machine and works.

The scraper and ingredient normalization are the pieces worth spending time on — they're where the actual complexity lives and where most of the value comes from. Everything else is plumbing.

Building with Claude Code handling the scaffolding meant I spent my time on the logic that matters: the normalization algorithm, the scraping fallback strategy, the data model decisions. The parts that require judgment rather than typing.

The app exists. I use it every week. That's the measure of whether it was worth building.
