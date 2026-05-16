+++
title = "Making All Caps as a Service"
description = "A walkthrough of building ACAAS, All Caps as a Service, from scratch as a demo docs site for Write the Docs, using Claude Code, Claude Design, and Mintlify."
date = "2026-05-12"

[taxonomies]
categories = ["Blog"]
tags = ["Claude Code", "developer-experience", "technical writing", "tooling"]

[extra]
subtitle = "I just needed something to demo against. It has rate limits now."
+++

For my recent [Write the Docs talk](../git-talk-recap/), I needed a docs site that felt real enough to demo against. I didn't have a convenient API lying around to document, so I built a toy API and a full docs site on top of that; overengineered, but did what I wanted and it was a fun side quest.

This is the full walkthrough of building ACAAS, All Caps as a Service. Looking back, I thought it was an interesting, abbreviated case study in docs tooling and building a site from scratch, so I wanted to jot down the decisions and steps.

## 1. Build the service

First, I used the Claude Code CLI to build a small API from scratch. I gave it the concept (you send text, it sends it back in all caps) and asked it to make it realistic enough to write docs against (like rate limits and error messages, and a few endpoints). It chose Python for the backend and came back with a few new endpoints like `/shout` and `/whisper`, and the supporting machinery to go with them.

I also told it I wanted one AI-powered feature, ideally something on the path to GA; a feature graduating from experimental is the kind of documentation moment I wanted to demo (and every SaaS company needs an AI feature). It proposed `/experimental`, which uses a small model to decide which words in your sentence deserve to be bolded.

After it finished building the API, I ran `/simplify`, which is [a bundled Claude Code skill](https://code.claude.com/docs/en/skills#bundled-skills) that does a quick cleanup pass to look for dead code and unnecessary indirection. This wasn't important for anything, but I wanted to at least feel like the code was cleaner than a one-shot pass. Then I [ran `/init`](https://code.claude.com/docs/en/best-practices#write-an-effective-claude-md) to generate a README and CLAUDE.md. The README is for future me, in case I ever need to run or modify this thing again. The CLAUDE.md is just good practice at this point, the scaffold I'd want if I came back to this project for something else.

## 2. Find a documentation platform

My first instinct for the docs platform was Zola or Astro. But before choosing, I did a sanity check on Mintlify, which I've worked with professionally for a couple of years and have never tried spinning up for a personal project. I'd assumed the free tier would be limited in some way that didn't fit a one-off demo site, but it wasn't. I created [acaas.mintlify.app](https://acaas.mintlify.app) in about a minute, cloned the repo, and was ready to go.

I was mildly embarrassed not to have known about the free tier, having worked with Mintlify for so long. I also kind of want to move my blog over to Mintlify now. The free tier comes with analytics, an editor, and a full agent tooling surface. I'd assumed at least half of that was paywalled.

![The Mintlify analytics dashboard for ACAAS, with 19 visitors and 41 views over a recent date range, a Humans/Agents traffic split, a Visitors Over Time bar chart, and a left sidebar covering Home, Editor, Analytics, Settings, and an Agents group for Workflows, Agent, Assistant, and MCP.](mintlify-dashboard.png)

## 3. Design the site

I didn't need to do this step at all, but it was the most fun. I have 0 design sense whatsoever, and [Claude Design](https://claude.ai/design) has been a lifesaver for me. I started a new prototype, described what I was building ("a Mintlify docs site for a fake API called All Caps as a Service"), and it started guiding me through the design steps until I had a lovely new branded docs site.

![Two Claude Design landing-page variations for ACAAS in dark mode, side by side: Variation A (Loud, Amber) with a large "send whisper. RECEIVE THUNDER." hero, and Variation B (Restrained, Cobalt) with a status dashboard, quickstart steps, and an endpoint table.](claude-design-landing.png)

I like that Claude Design asks clarifying questions as it works; things like what kind of feel I was after (bold or minimal or standard SaaS), whether I had color directions or wanted it to pick, etc. I landed on a theme I was happy with, had it generate a favicon and logo to match, and the whole time it was working inside Mintlify's design constraints rather than producing something I'd have to retrofit. When I was done, I chose the "Handoff to Claude Code" export option and went on with my day. 

![Claude Design output for ACAAS branding: eight candidate icon marks (Monogram A, Sound bars, Bang, Stacked carets, Megaphone, aA transform, Boost, Chevrons) with short rationales, three wordmark lockups, and a recommended pick of the aA transform with sizing and dark-mode previews.](claude-design-marks.png)

## 4. Apply the design

I used Claude Code Desktop for this because it has a built-in local preview. I could run the site locally and watch it render while Claude made changes, and Claude can see the preview too, so it catches its own rendering errors without me having to describe what I'm seeing. The workflow felt closer to pairing than to instructing. You could also do this with the CLI and then use the [Claude in Chrome extension](https://code.claude.com/docs/en/chrome) or [Playwright MCP](https://github.com/microsoft/playwright-mcp) to have Claude iterate on the design itself locally, but I just like the Desktop in-app experience.

![Claude Code Desktop with the conversation in the left pane (a turn confirming that docs.json, index.mdx, styles.css, and new logo files were copied to the worktree, with Mintlify hot-reloading on localhost:3000) and the live ACAAS docs site rendered at localhost:3000 in the right pane, showing the freshly-applied "send whisper. RECEIVE THUNDER." hero.](claude-code-desktop-preview.png)

This step was pretty anticlimactic in a nice way. Claude took the design handoff, implemented it cleanly, and there wasn't anything to debug. As an aside, I've shipped so many ugly, half-finished prototypes and waved it away with "I don't like CSS," and it feels like the days of that are finally over.

## 5. Scaffold the docs

I let AI generate the content with little direction, and I feel very fine about that. I said I wanted getting-started guides, cookbooks, and a changelog, and it produced them. They're not in my voice, none of the cookbook recipes are tested, and I'm sure a few are confidently wrong in ways that don't matter for a demo.

The thing that surprised me was that new Mintlify sites ship with a pre-built `AGENTS.md` file and a `.claude/` directory with settings and skills scaffolding for working with Mintlify, along with some basic style guides. Claude Code already understood how Mintlify projects were structured before I told it anything.

## 6. Create an OpenAPI spec

The last piece was an API reference section. This is one of my favorite Mintlify features, even though the docs I work on now don't have an API component. The API reference pages are built in and include an interactive playground where visitors can make live calls to the service from inside the docs themselves. All it needs is an OpenAPI spec, which Claude Code generated directly from the service code.

---

Start to finish, the whole build was about two hours, most of which was the design step and a small detour deciding how much realism the rate limits needed.

What I kept noticing was how quiet the handoffs between tools were. The Claude Design export plugged into Claude Code without me reformatting anything. The Mintlify scaffolding meant I didn't have to onboard Claude Code to the project separately. The OpenAPI spec came from the code rather than being written from scratch. Every layer had a tool that fit it. A year ago I would have spent the same two hours scaffolding just the initial API and called it a productive afternoon.

End result: <!-- FLAGGED: fragment, no subject — consider "I used this in my talk..." or restructure --> used this in my talk [the git commands I avoided for nine years](../git-talk-recap/).