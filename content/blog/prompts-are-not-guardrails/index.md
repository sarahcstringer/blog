+++
title = "Prompts are not guardrails"
description = "Why telling an AI agent not to do something isn't the same as making sure it can't, and where those constraints can live instead."
date = "2026-09-26"

[taxonomies]
categories = ["Blog"]
tags = ["AI", "Claude Code", "hooks", "developer-experience", "guardrails"]

[extra]
subtitle = "The problem isn't that it did the thing, it's that it was even possible to do the thing."
+++

My husband likes to say "prompts aren't guardrails," and it's been ringing in my head for months, so I figured I'd finally write down my take on it.

I regularly see people who are frustrated, or kind of confused, because they told an LLM very emphatically (in ALL CAPS! Repeatedly! Very seriously!) to never do something... and then it did the thing anyway. This shows up in AGENTS.md lines like:

- **ALWAYS FOLLOW THE STYLE GUIDE!**
- Never make an unsigned commit.
- Do NOT push to main. NEVER PUSH TO MAIN.

I've also winced my way through stories of LLMs deleting full production databases or wiping someone's home directory. The frustration or anger when an LLM does the wrong thing makes sense -- we told the computer what not to do, gave it examples of the right thing, and clearly stated how bad it would be if it disobeyed.

But, at the end of the day, the model is a probabilistic token generator with a ton of other stuff in its context window. If something really needs to happen or to not happen, more words and rules aren't the solution. The rule has to live somewhere that can actually stop the model, which is what "prompts aren't guardrails" means to me.

## Imagining myself doing the LLM's task

I don't want to anthropomorphize models, but sometimes imagining myself in a similar situation to the LLM, or trying to do the same task, helps me see a little why something might or might not work. (It's how I realized that [onboarding a human made my AI smarter](https://sdeaton.com/blog/onboarding-human-made-ai-smarter/); if nobody gave me onboarding materials, I wouldn't understand how to do the job, and I shouldn't expect differently of an LLM.)

Here's what I'm imagining in the "prompts aren't guardrails" scenarios:

Say you asked me to edit a doc, and then handed me a 20-page style guide plus the Chicago Manual of Style and told me to follow all the rules exactly. I can guarantee I would **not** follow them exactly and would miss a ton of rules. I'd read it and I'd try my best, and I'd still miss things.

If you gave me a list of the most important rules that you cared about, I'd probably get closer to respecting those, but might still drop a few items.

If you instead gave me a tool that tells me every time I've violated the style guide, and asked me to get that count to zero, I could do that. The tool does the remembering, and my job becomes fixing whatever it flags.

And if you set things up so that I couldn't even merge a doc until the style guide violations count was at zero, then it would definitely work out, because the doc has no other way to ship.

The first setup depends on me remembering; the second helps me at least prioritize what to remember; the third tells me when I forgot; and the fourth makes it impossible to ship what I forgot. Only that last one is what I'd call a guardrail, and it's what's missing when the model doesn't obey what you told it to do.

## If only words stop it, there isn't a guardrail

The same thing applies for actions the LLM (or I) should or shouldn't take. If something should never happen, the system needs to actually prevent it from happening.

Say you don't want me to SSH into production, but it's technically possible; I have the keys and nothing is stopping me except the fact that the CTO said everyone needs to seriously stop yeehawing into prod (this may or may not be a real example).

Most days, I'd respect that. However, in the one case where something is broken on prod and SSHing in to fix it would be the fastest path, the rule is something I weigh against the outage and might decide to ignore. The rule and its words, however strongly emphasized, are not the guardrail. If the CTO really wanted me to not do something, the system should have enforced that rather than relying on a memo.

### Why was that even allowed?

In [blameless postmortem cultures](https://sre.google/sre-book/postmortem-culture/), the goal is to look for the system failure rather than blaming an individual person for their choices. If something broke or something bad happened, why was that thing allowed to happen at all?

If someone on the team ran `terraform apply` and accidentally took down our whole EC2 cluster (another possibly real life example), the useful question is why anyone was able to do that in the first place, and not why that individual wasn't more careful with reading the terraform output in this one instance.

This is something that could apply to LLM failures as well. Rather than asking the LLM why it didn't do something you'd asked it to, it might be better to fix it at the root. This matters even more for a model than for a person. After an incident, a person usually remembers what happened and does better next time, but the model comes in fresh every session. Anything it "learns" from being corrected only sticks if it gets written down somewhere, which puts it right back in words.

If it's something small that's more of an annoyance (ex: why didn't you remind me when the job finished running?), it might be worth rewording your skill or prompt a bit and trying again. But everything you actually care about needs to move out of words and into a deterministic check or a system that doesn't let it happen at all. 

### Less convenient, but safer

Guardrails can make things less convenient. Not being able to SSH into prod is annoying, especially when something is broken and you know exactly what to fix. Branch protection means even a one-character typo fix goes through a pull request. Network restrictions mean the model won't always have everything I might want in its context, but it also isn't going to send internal data to a random site.

The small annoyances of guardrails add up to a lot fewer incidents. I sometimes hate the extra work I have to do because of them, but ultimately respect and appreciate that we have systems that prevent the "definitely don't do this" actions. And often the guardrails are built from accidents of the past.

There are ways to make that tradeoff less painful, too. Grant only what the task needs, like a read-only database user for a model that only has to query data, and you keep most of the convenience without the risk of it dropping a table.

## Where guardrails can live

If you're regularly running into issues with a model ignoring what you're saying, there are lots of places where you could start looking to address root failures outside of the model.

Most of the ones I rely on in my own work are pretty standard CI. Branch and merge protection keep anything from landing on main without a review and passing checks. [Vale](https://vale.sh/) checks the style guide rules that can be pattern-matched, [lychee](https://lychee.cli.rs/) and [`mint broken-links`](https://www.mintlify.com/docs/cli/commands#mint-broken-links) catch broken links, and another check looks for anything private that shouldn't end up public. Here's where those, and the examples from the top of this post, could live.

| If you want... | Put the guardrail in... |
|---|---|
| No pushes to main | Branch protection with required reviews and passing checks, plus a Claude Code [permission deny rule](https://code.claude.com/docs/en/permissions) or `PreToolUse` hook that blocks the push before it happens |
| The style guide followed | Vale in CI as a required check, plus a Claude Code `PostToolUse` [hook](https://code.claude.com/docs/en/hooks-guide) that runs it after every edit so problems show up before CI does |
| No broken links | lychee or a similar tool in CI as a required check |
| Nothing private made public | A required CI check that scans for secrets and internal info before merge, plus push protection |
| Only signed commits | Branch protection that requires signed commits |
| A task that runs every week | A scheduled job, like a cron job, a GitHub Actions schedule, or a Claude Code routine |

For anything running unattended, a sandbox or container bounds what can go wrong to whatever that environment can reach. Network restrictions go a step further by keeping it off networks it has no business touching, like sensitive internal systems or sites you don't trust.

### Side benefit: more room in the context window

Doing some of these can also have the benefit of removing context for the LLM. Instead of spending the context window describing what not to do, just don't let the bad things happen, and save that space for the things you actually want to tell the LLM.

I used to have our whole style guide in skills and CLAUDE.md, including tons of lines describing deterministic, mechanical rules with examples of right and wrong usage. After adding Vale, I was able to remove all of those. The rules still exist in the Vale config, where they run on every change, and the model only hears about the specific ones it broke. Now the model can focus its attention on the work that takes judgment instead of trying to remember capitalization rules.

## Words still matter

None of this means you should stop telling the model what you want. Written instructions are still how it knows what it's supposed to do, learns your preferences, understands team conventions, etc. But anything you put into words is something the model considers, the same way I'd consider the CTO's memo about yeehawing into prod when things were on fire. If something truly can't happen, all the ALL CAPS in the world won't do what a branch protection rule does.