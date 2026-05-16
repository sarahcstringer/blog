+++
title = "Prompts are not guardrails"
description = "On the difference between asking Claude not to do something and actually making sure it can't."
date = "2026-05-12"
draft = true

[taxonomies]
categories = ["Blog"]
tags = ["Claude Code", "hooks", "developer-experience"]

[extra]
subtitle = "I kept putting guardrails in prompts. Prompts are not where guardrails go."
+++

I feel bad any time someone posts about an AI agent doing something destructive they couldn't reverse. But I keep coming back to the same thought in response, which is that prompts are not guardrails. If there's something that should never happen, there are actual places to put that constraint, and the system prompt is not one of them.

The distinction between asking and enforcing matters more than it might seem at first. When you write "never delete files outside this directory" in your CLAUDE.md, you're asking the model to follow that rule. It usually does. But the model reads that instruction the same way it reads everything else in context, as something to weigh and work with rather than a hard boundary. When the context window fills up and the model starts managing what to keep in focus versus what to deprioritize, long-ago system prompt instructions are among the first things to slip. And even in a fresh context with plenty of room, a sufficiently unusual situation can send the model somewhere your prompt didn't anticipate.

A rule the model might not follow under pressure is not a guardrail. It's a suggestion you got lucky with so far.

## Why prompts fail as guardrails

A language model is not a rules engine. It's good at following instructions most of the time, which is exactly why it feels like the right place to put behavioral constraints. But "most of the time" is not the bar you want for something that should never happen. The model is nondeterministic by design, and what gets prioritized shifts based on context, conversation length, and what else is happening in the session. Putting a hard rule in a prompt is a bit like writing "do not enter" on an unlocked door: it works on most people, most of the time, which is a terrible guarantee for the cases where it matters.

For things that should not happen, you need enforcement at a layer the model can't reason its way around.

## Hooks

The right tool for "this should never happen" in Claude Code is a `PreToolUse` [hook](https://code.claude.com/docs/en/hooks-guide). Hooks are shell commands that fire on Claude Code lifecycle events, and `PreToolUse` runs before any tool call. A hook that exits with a non-zero status stops the tool call and tells Claude why it was blocked. The model doesn't get to weigh in on whether the rule applies. A shell script returning 1 is not something Claude can interpret its way past.

The kinds of things worth putting here: blocking destructive commands outside a scratch directory, preventing writes to certain files, requiring confirmation before anything touches production config. The principle is that if the cost of the model getting it wrong once is too high, a hook is where the constraint belongs.

## Permissions

Claude Code's permissions layer also operates outside the model. You can restrict which tools are available, which directories can be read or written, and which shell commands can run. For read-only contexts like auditing an unfamiliar codebase or investigating a problem without any risk of changing anything, setting the permission mode to `read-only` is cleaner and more reliable than asking Claude to please not modify files. In read-only mode, it can't.

## Sandbox

For automation running unattended, or anything touching infrastructure you care about, a sandboxed execution environment is worth the setup cost. Running Claude Code in a container means that even if something goes wrong in a way you didn't anticipate, the blast radius is bounded by the environment rather than by how well your CLAUDE.md held up under pressure. This isn't a Claude-specific point. Any agent running shell commands in an unsupervised loop deserves a constrained environment.

## Schedules

I saw someone post a frustrated observation that Claude hadn't run a task they'd asked it to run every week. The ask was in a prompt. Nothing in a prompt has memory across sessions since each conversation starts fresh, which means a weekly task described only in-context has nothing to trigger it. If something needs to run reliably on a schedule, it needs to live somewhere durable.

Claude Code has [remote routines](https://code.claude.com/docs/en) that run on a cron schedule outside any interactive session. A GitHub Actions workflow is another solid option. The specific tool matters less than the underlying point, which is that the schedule has to exist somewhere outside the model's context for it to be real. A prompt saying "remember to do this every Tuesday" is not a scheduler.

## Linters

The same reasoning applies to style and content rules. A prompt saying "never use passive voice" will miss things sometimes, because the model is nondeterministic and what it prioritizes shifts with context. A deterministic linter like [Vale](https://vale.sh/) doesn't miss things, because it's not weighing intent or managing a context window. It pattern-matches against rules that either fire or don't.

For anything you need to catch consistently, a linter is more reliable than a prompt. If you have forbidden phrases, required heading formats, capitalization conventions, or prose rules that matter, a prompt handles the general case but a linter gives you real coverage on the tail. Use both if you like, but don't use only the prompt and call it a guardrail.

## Other prompt-shaped things worth knowing about

Two things that do work well as more durable instructions, as long as you understand what they are.

[Skills](https://code.claude.com/docs/en/skills) are loaded every time they're triggered, which makes them more reliable than hoping a CLAUDE.md rule is still active in memory when you need it. If you find yourself adding the same correction across multiple conversations, that pattern belongs in a skill that fires in the relevant context rather than a standing rule that might not be in focus when it matters. Skills are not enforcement, but they're considerably stickier than in-context instructions.

[Output styles](https://code.claude.com/docs/en/output-styles) replace part of the system prompt on every session startup. If the model keeps responding in a way that bothers you, too verbose, too formal, wrong register entirely, an output style is the right place to fix it. You write the style description once and it's active every session by default. Again, not enforcement in the way hooks are, but more durable than an in-context instruction, and knowing that distinction saves a lot of frustrated re-prompting.

The thing I find myself thinking whenever I see a post about a rogue agent is that the model did what models do: it followed instructions probabilistically, in a context that eventually drifted far enough from the instruction that the instruction stopped holding. The useful question after something like that isn't just "what should I have told it" but "where should this constraint actually live." For a lot of things, the answer is outside the conversation entirely.
