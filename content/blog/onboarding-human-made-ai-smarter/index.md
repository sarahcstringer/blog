+++
title = "How onboarding a human made my AI smarter"
date = "2025-09-21"
description = "Apparently neither humans nor AI can read my mind?"
[taxonomies]
categories = ["Blog"]
tags = ["AI", "documentation", "onboarding", "context"]
+++

The most recent team I joined had a set of docs but had never had a dedicated technical writer. It was a collection of information that had grown organically over time without style guides, linting, or any consistent voice.

I had every intention of coming in and setting up the foundational guides and best practices in my first month. But then I got sucked into fighting fires and pushing out new docs. There was always something more urgent or rewarding than setting up those guardrails. Plus, I already knew the style I wanted and was the only one who had to conform to it.

Six months later, a new person joined our team, and I realized I had nothing set up for her to onboard onto the docs. I became the bottleneck, dropping nitpicky PR comments like *"*actually, we use sentence-style capitalization…*"* instead of setting her up to succeed.

I scrambled to codify our standards, and was surprised with the effect: getting those base guides in place didn't just help my human teammate, but it elevated my AI tools, too. [CodeRabbit](https://www.coderabbit.ai/) reviews became so on point and [Cursor](https://cursor.com/) generated drafts in our correct voice following the correct templates.

I’d heard the term “context engineering,” but it wasn’t until I reflected on this process that it clicked: whether human or AI, teammates need context. The same onboarding materials that help humans understand how we do things here also give AI the information needed to perform at a higher level.

## What I did

Here are the pieces I put in place that made a difference:

### Style, voice, and tone

The biggest lift was that I finally wrote down all the style rules I'd been carrying in my head.

- I set [Google’s](https://developers.google.com/style) as the base style guide to follow, so I didn’t have to document every single piece

- I wrote down any overriding preferences for capitalization, terminology, and phrasing with real before/after examples

- I stored everything in a `STYLE-GUIDE.md` for a human-readable format and in `.cursor` [project rules](https://cursor.com/docs/context/rules) so my AI tools could reference them directly

With this in place, Cursor was able to almost one-shot new doc generation and CodeRabbit, which also can access Cursor project rules, started giving truly helpful PR reviews that saved me from my own carelessness many, many times.

### Linting and automation

I finally set up [Vale](https://vale.sh/) to catch any spelling or style errors before they reached production:

- I used AI to help me tune Vale's rules to cut down false positives while catching real problems

- I looked at established projects that use Vale like [Grafana](https://github.com/grafana/grafana) to help figure out best practices

- I configured Vale to flag the specific things that matter to our docs, like terminology mismatches, formatting quirks, tone shifts

- I squashed all the errors in our docs so we had a baseline and then added Vale to our CI to catch any new errors

Now Cursor is able to see and fix Vale errors as it edits docs, and the human author gets notifications about errors as PR comments, so we don’t introduce new ones.

### Process documentation

I wrote down not just what I prefer, but how I actually work.

- I documented my editorial workflow: step-by-step how I approach drafts and reviews, what I look for first

- I captured context about our audience and goals that I'd never articulated before

- I created templates that reflect how I actually structure different types of docs

- I pointed Cursor to the source of truth repos so it could verify information from the source like I would

Now Cursor and CodeRabbit are able to look holistically at the docs I create and consider additional context the audience might need when approaching them.

## What I learned

I didn't expect AI to get better just because I onboarded a human teammate, but once I wrote down our rules and automated our guardrails, AI started performing at a higher level, too.

I hadn’t been treating AI like a collaborative partner who needs the same foundational knowledge I’d give a person. I’d just kind of hoped AI would figure out my preferences and get it together eventually. I now give it the explicit context so AI can make smart, independent decisions.

I built the context once. Now future AI tools and future colleagues can hit the ground running.
