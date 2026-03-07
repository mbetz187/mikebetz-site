---
title: "I Built This Site With an AI Pair Programmer. Here's What Actually Happened."
description: "A honest account of using Claude Code alongside VS Code, GitHub, Netlify, and Cloudflare to get a site from zero to live."
date: 2026-03-07
tags: ["AI", "Tools", "Build Log"]
---

I didn't want to spend two weeks setting up a blog. I've done that before — picking the perfect stack, customizing everything, and never actually writing anything. This time I wanted to move fast, ship something real, and learn in the process.

So I used an AI pair programmer — Claude Code — to help build this site. Not to hand the work off. To move faster and make better decisions.

Here's an honest account of how it went.

## The Stack Decision

I came in with some constraints. I wanted:

- Static site generation (fast, cheap, no server to babysit)
- Markdown-based content (write posts like a developer, not in some CMS)
- Easy deployment with zero ongoing maintenance burden
- A real domain with proper DNS handling

Claude helped me evaluate options without the usual analysis paralysis. We landed on **Astro** as the framework — it's purpose-built for content sites, has excellent Markdown support through its content collections API, and generates clean static output. **Tailwind CSS** handled styling, with the typography plugin giving blog content sensible defaults without me hand-crafting prose styles.

That decision took about ten minutes. It would have taken me two days alone.

## The Toolchain

**VS Code** was the development environment. Claude Code integrates directly into the terminal, so the workflow was natural: edit in VS Code, talk to Claude in the terminal, iterate. No context switching to a browser-based chat window.

**GitHub** handled source control. Every change committed and pushed — not because I was precious about version history on a personal site, but because it's the right habit. It also unlocked the deployment workflow.

**Netlify** connected directly to the GitHub repo. Every push to main triggers a build and deploy automatically. Zero configuration after the initial setup. The site is live within seconds of a commit.

**Cloudflare** sits in front of everything handling DNS and CDN. The domain is routed through Cloudflare, which means fast global edge caching, free SSL, and a layer of protection without any additional work on my end. It's the kind of infrastructure decision that's easy to overlook on a personal project and worth getting right from the start.

## What the AI Pair Programming Actually Looked Like

Here's what I want to be clear about: I didn't describe the site and have Claude generate it. That's not how good software gets built.

What I did was direct the work. I made the architectural decisions. I reviewed every file. Claude handled the implementation — scaffolding the Astro project, wiring up the content collections schema, configuring Tailwind, setting up the blog post routing. The things that are tedious to do by hand but don't require deep judgment calls.

When I had questions — how does Astro's content collection API handle frontmatter types? what's the right Netlify config for a static Astro build? — I got answers immediately, in context, without leaving my editor. That's the real productivity gain: not automation, but elimination of friction.

## What I'd Do Differently

A few things I'd change on the next build:

**Start with the content schema.** I had a rough idea of what my blog posts would contain, but formalizing the TypeScript schema earlier would have saved a small amount of backtracking. Define your data model first.

**Set up the Cloudflare/Netlify connection before you touch code.** Getting DNS right can take time to propagate. Do it early, let it settle, and you're not blocked at the end waiting on DNS when you want to go live.

**Commit more granularly.** I batched some early changes into large commits. Fine for a personal site, but a habit worth fighting against.

## The Honest Take on AI-Assisted Development

AI pair programming is genuinely useful. It's not magic, and it's not a replacement for knowing what you're doing. If I didn't have a clear picture of what I wanted to build and why, Claude couldn't have helped me build it well.

What it does well: implementation speed, answering specific technical questions, catching things you'd otherwise miss, and keeping momentum when you'd otherwise stop to look something up.

What it doesn't replace: judgment, architecture decisions, knowing when something is over-engineered.

For a project like this — defined scope, clear tech choices, solo developer — it was the right tool. I went from nothing to a deployed site with real content in a fraction of the time it would have taken otherwise.

More importantly: the site exists. The writing is happening. That's the actual goal.
