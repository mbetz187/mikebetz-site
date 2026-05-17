---
title: "Site Fixes, Performance Wins, and What I Learned Debugging CI With AI"
description: "A behind-the-scenes look at a full site audit session: image optimization, self-hosted fonts, SEO fixes, accessibility improvements, and two CI build failures I had to hunt down and kill."
date: 2026-05-17
tags: ["Build Log", "Performance", "AI", "Tools"]
---

I ran a full audit session on this site recently. What started as "let me check a few things" turned into a few hours of performance work, SEO cleanup, accessibility fixes, and chasing down a pair of CI build failures that were more interesting than they had any right to be.

Here's the full breakdown.

## Image Optimization: The Easy Win Nobody Does

The og-image and profile photo were each sitting at 2.2MB. That's embarrassing for a static site. No excuse for it.

First pass: compress them. Got the OG image down to 519KB and the profile photo to 131KB using standard compression. Better, but not done.

Second pass: convert to WebP and let Astro handle it properly. Switched the profile photo to use Astro's built-in `Image` component, which generates optimized WebP at build time. Final size for that image: **6KB**. Down from 2.2MB. That's a 99.7% reduction. The Astro Image component is genuinely impressive once you stop fighting it and just use it correctly.

Added responsive `srcset` attributes too, generating four sizes (192px, 224px, 384px, 448px) so retina displays get crisp images without serving 4x the bytes to everyone else.

## Font Loading Was Blocking Render

The site was pulling fonts from Google Fonts. That creates a render-blocking request to a third-party server before the page can paint anything visible.

The fix was to self-host the fonts instead. Download the font files, put them in `public/fonts/`, and serve them yourself. Then use `font-display: swap` so text renders in a fallback font immediately while the custom font loads in the background. No more blank screen waiting on a DNS lookup to someone else's CDN.

One wrinkle: I initially tried deferring the Google Analytics script too, thinking it would help. It did not. It spiked Total Blocking Time instead, which is the opposite of the goal. Reverted that immediately. The lesson is that `defer` on a script doesn't mean "run later and stop blocking" -- it means "parse later but still execute on the main thread before the page is interactive." For GA specifically, the standard `async` load is the right call.

## SEO: Fixing What Was Already Broken

Found an OG image bug -- the meta tag was pointing to the wrong path, so social media previews were broken. Fixed that and added proper `Article` structured data schema to blog posts so Google can understand what it's looking at.

Added an RSS feed too. It's a small thing but it matters -- a meaningful percentage of technical readers still use RSS, and an Astro-generated feed is about 20 lines of code to set up.

Meta description and canonical URL handling got a cleanup pass while I was in there.

## Blog UX Features

Added three things that should have been there from the start:

**Reading time.** Calculated from word count at build time, displayed at the top of each post. Sets expectations for the reader.

**Social share buttons.** One-click sharing to Twitter/X and LinkedIn. I'm not going to pretend social sharing doesn't matter for a blog trying to build an audience.

**Tag filtering on the blog index.** You can now filter posts by topic without leaving the page. Client-side filtering, no page reload.

## Accessibility: Footer Contrast Was Failing

Ran the site against accessibility contrast requirements and the footer was failing. The text color wasn't meeting the WCAG minimum contrast ratio against the background.

This one was a specificity issue. The footer was inheriting a text color that looked fine in isolation but failed when you put it against the actual footer background. Fixed it by being explicit with the color class on each footer element rather than relying on inheritance.

Also added an author bio component to the bottom of blog posts. Useful context for anyone who lands on a post directly from search and doesn't know who they're reading.

## The CI Failures (The Interesting Part)

This is where it got complicated.

**Failure 1: ENOENT on dist/fonts**

The GitHub Actions build was failing with a file-not-found error on `dist/fonts/`. What was happening: Astro's build step expected to copy font files into `dist/fonts/`, but the directory had to exist before the copy command ran. On the local machine this worked fine because the directory persisted between builds. On CI, every run starts clean. The fix was to pre-create the `dist/fonts/` directory as a build step before running the Astro build.

**Failure 2: Garbled filename in the git index**

This one was stranger. A previous bash command had used its own command text as a filename -- meaning a long string of path text got created as an actual file in the repository and committed to git. On CI, when git tried to restore that file, it threw an error because the path wasn't valid in the CI environment.

The fix was to remove the file from the git index entirely (`git rm --cached`). No file content to delete, just an index entry pointing to something that shouldn't exist.

Both failures are good examples of the "works on my machine" category of bugs. The local environment had state that CI didn't have, and the CI environment surfaced assumptions I hadn't realized I was making.

## Platform Clarification: GitHub Pages, Not Netlify

Somewhere along the way the repo had a `netlify.toml` configuration file that didn't belong there -- the site is deployed on GitHub Pages, not Netlify. Removed it. The Netlify cache header configuration it contained was irrelevant and potentially confusing.

## Using Claude Code for All of This

I ran the entire session with Claude Code handling implementation. The workflow was: I'd identify what needed fixing, describe the problem and expected behavior, review the change, iterate if needed.

What worked well: the CI failures in particular. Debugging CI failures is tedious because the feedback loop is slow -- push, wait for the runner, read the log, push again. Having Claude Code read the workflow file, trace the error back to its cause, and propose a fix meant I was pushing targeted changes rather than guessing. The garbled filename issue especially -- I wouldn't have thought to look in the git index for that.

What required my judgment: the GA defer revert. Claude implemented what I asked for, and it was wrong. Not wrong in the sense of bad code -- wrong in the sense that I misunderstood how `defer` would interact with Total Blocking Time. I had to recognize the regression from the metrics, understand why it happened, and decide to revert. That's not something you can delegate.

The session probably would have taken twice as long without the AI pair programming, and I'd have spent more of that time on boilerplate and less on the actual decisions. The ratio of thinking to typing improved.

## What's Next

The site is in good shape technically. The next priority is content -- more posts, more consistently. A technically perfect empty blog is just an empty blog.
