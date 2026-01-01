+++
title = "Claude Code beyond prompting: What I learned in my first three weeks"
description = "Turns out I'd been using a Swiss Army knife with only the main blade"
date = "2025-12-31"

[taxonomies]
categories = ["Blog"]
tags = ["AI", "Claude Code", "technical writing", "tooling", "workflows"]
+++

Before I joined Anthropic three weeks ago, I thought I was pretty good at Claude Code.

At my previous job, I used it constantly. I'd put our entire style guide into a `CLAUDE.md` (philosophy, voice, grammar rules, repo structure, the works), and that alone felt like a cheat code. Suddenly every edit Claude made followed our conventions automatically. No more reminding it about sentence case headings or active voice.

Adding AI to my technical writing workflow felt like a huge accelerant. I was generating diagrams I couldn't make myself, getting instant consistency checks, automating the tedious parts of doc reviews. I'd heard people mention subagents and skills and hooks, but I didn't quite get how they fit in. What I was doing worked great, so I just kept prompting.

Then I joined Anthropic to write documentation for Claude Code, and within the first week I realized: I'd been using a Swiss Army knife but only the main blade. There were scissors, a screwdriver, and a tiny saw folded in there the whole time.

I'd stumbled into using one powerful feature (`CLAUDE.md`) and assumed that was it. Turns out there's a whole ecosystem: slash commands you can save, subagents you can configure and reuse, hooks that run automatically, plugins people have already built. I just had no idea any of it existed.

This post is about the tools I didn't know I was missing, and how they changed my technical writing workflow once I started using them intentionally.

So here's what I've been learning. What Claude Code can actually do for technical writing workflows (among many others), way beyond "put your style guide in `CLAUDE.md` and start prompting."

## Where I started (and where you might be too)

The `CLAUDE.md` file was my first big unlock. If you haven't done this yet, start here. It's the highest-impact, lowest-effort improvement you can make.

Mine had everything: our documentation philosophy, voice and tone guidelines, grammar rules, formatting conventions, repo structure, even notes about our publishing workflow. Claude reads this automatically at the start of every conversation, so every edit followed our conventions without me having to remind it.

The tradeoff: since `CLAUDE.md` loads fully at the start of every conversation, every byte counts toward your context limit from the beginning. This makes it perfect for universal guidelines that apply to everything, but not ideal for specialized knowledge you only need occasionally.

Here's a taste of what the voice section looked like:

```markdown
## Voice and tone

Readers are busy software engineers solving problems. Be direct, practical, and respect their time.

- Address the reader as "you"
- Use active voice
- Lead with what the reader can DO, not what the product IS
...
```

I'd also connected some frontend MCP servers so Claude could actually look at my local preview and catch visual issues. I'd even tried a few plugins for code quality checks.

That got me really far. If you're doing this much, you're probably already ahead of most people using AI for docs work.

But here's what I didn't realize: my `CLAUDE.md` was doing a lot of heavy lifting that could be broken out into more specialized tools, and I kept rewriting tasks I wanted to perform frequently. There was also a whole layer of capability I wasn't touching at all.

## What else Claude Code can do (that I wasn't using)

Here are the pieces I'd heard about but never actually tried, and what clicked once I did.

**Quick summary:**
- [`CLAUDE.md`](#where-i-started-and-where-you-might-be-too): Universal guidelines (loads every conversation)
- [Slash commands](#slash-commands-one-prompt-saved-forever): Saved prompts you invoke by name
- [Skills](#skills-context-that-loads-only-when-you-need-it): Specialized instructions that load on-demand
- [Subagents](#subagents-parallel-actions-without-losing-focus): Parallel work with isolated context
- [Hooks](#hooks-automation-that-runs-without-you-asking): Automatic checks/actions
- [Plugins](#plugins-bundle-it-all-up-and-share-it): Bundle it all and share

Start with `CLAUDE.md`, add the others only when you feel friction.

### Slash commands: One prompt, saved forever

I'd used a few slash commands before, but I wasn't thinking about them systematically. Turns out I should have just reviewed the list of built-in ones, because there's a lot I was missing; things like `/rename` to clean up session names and `/init` to get a project off the ground. So good.

Beyond the built-ins, any prompt you type more than twice could really just be a custom command. They're Markdown files you store in `.claude/commands/`.

Unlike `CLAUDE.md`, slash commands don't load into context until you invoke them. Only their descriptions are available to Claude through a character-budgeted tool. This means you can have dozens of commands saved without cluttering your context.

When you do invoke a command, it runs with your full project context: `CLAUDE.md`, conversation history, everything. You're just injecting a reusable prompt into the main agent. Compare this to subagents (covered below), which only get the specific context you pass to them.

Here are two examples from a technical writing toolkit:

**`/review`**, a full doc review checklist:

```markdown
Review this document for:
1. Style guide compliance (check CLAUDE.md)
2. Technical accuracy (do the code samples work?)
3. Broken links and references
4. Missing context (would a new user understand this?)
5. Structural issues (headers, flow, completeness)

Categorize issues as: blocking, should fix, nice to have.
```

**`/simplify`**, for when you've written something too dense:

```markdown
Simplify this content for a developer who's new to the product.
- Remove jargon or define it on first use
- Shorten sentences
- Cut anything not essential to the core concept
- Add an example if it would help
```

You can run these with arguments: `/review en/quickstart.mdx` or `/simplify` with selected text.

Pick one or two commands that solve your biggest friction points. You don't need a dozen to start getting value.

([Full slash commands documentation](https://code.claude.com/docs/en/slash-commands))

### Skills: Context that loads only when you need it

Having all my style guide information in `CLAUDE.md` was super useful, but it turns out I was using it as a hammer for every problem I encountered, when there were more appropriate tools available.

One issue with a giant `CLAUDE.md` file: Claude has a context limit. The more you add, the more likely something gets dropped, or Claude gets confused about what's relevant right now.

Skills are one way to solve this. They're directories with instructions, templates, and reference files. At startup, Claude only loads skill names and descriptions. The full `SKILL.md` file only loads when you (or Claude) trigger that skill and you confirm. This is way more efficient than putting everything in `CLAUDE.md`.

For technical writing, you can set up skills for different doc types:

**API documentation skill:**

```
.claude/skills/api-docs/
├── SKILL.md           # Instructions for API doc structure
├── template.md        # Standard template with all sections
└── examples/
    ├── good-example.md    # "Here's what we're going for"
    └── common-mistakes.md # "Avoid these patterns"
```

**Tutorial skill:**

```
.claude/skills/tutorials/
├── SKILL.md           # How we structure tutorials
├── template.md        # Intro, prerequisites, steps, next steps
└── voice-examples.md  # Examples of our tutorial tone
```

The supporting files (templates, examples) only get loaded when the task actually needs them. Claude discovers them through links in your `SKILL.md`, so you can keep detailed reference material available without it eating up context until it's relevant.

The `SKILL.md` file tells Claude when to use this skill and how:

```markdown
# API Documentation Skill

Use this skill when writing or editing API reference documentation.

## Structure
Every API doc needs: Overview, Authentication, Endpoints, Error codes, Examples

## Voice
More formal than tutorials. Assume reader knows HTTP basics.

## Template
See template.md for the standard structure. Don't skip sections.
```

Now your `CLAUDE.md` can stay lean with only universal guidelines for your project, while specialized context loads automatically when you need it.

([Full skills documentation](https://code.claude.com/docs/en/skills))

---

**A note on Skills vs MCP:** I've heard people say that Skills make MCP servers less useful, but that's not the case. They serve different purposes. **MCP is the connection to your data** (Slack, GitHub, Google Drive). **Skills are the instructions for how to use that data.**

For example:
- MCP server connects Claude to your GitHub issues
- Skill says "When reviewing docs, check issues labeled 'documentation' for common user confusion"

You can have MCP without skills (you just prompt every time), but skills paired with MCP are way more powerful. Skills turn "I have access to this data" into "here's when and how to use it."

([MCP documentation](https://code.claude.com/docs/en/mcp))

---

### Subagents: Parallel actions without losing focus

You might notice some overlap here. Couldn't `/review` also be a subagent? Yes. The main difference is context management.

- **Slash command (`/review`)**: Runs in your main conversation with full context. Good when you want Claude to consider the entire discussion and project state.
- **Subagent (review-agent)**: Runs in isolation with only the context you pass it. Good when you want to run multiple reviews in parallel or keep the main conversation focused.

You can have both. Use the slash command for quick, context-aware reviews. Use the subagent when you're juggling multiple tasks at once.

This is the feature I'd heard a lot about but never really understood. Subagents are separate Claude sessions that can run in the background with their own context. The main agent can inject specific context into them, and they report back when they're done.

At first, I thought subagents were just something you spawn ad hoc. You can totally do that. Ask Claude Code to spin up a subagent for a one-off task and it will. That got me pretty far.

But here's what took me longer to figure out: you can *save* subagent configurations. They're Markdown files with YAML frontmatter that live in `.claude/agents/`. Once configured, they become reusable tools you can invoke by name.

This is where it gets useful for technical writing: the end-of-doc review gauntlet. Subagents run in their own isolated context. They only see what the main agent explicitly passes to them. This means you can kick off three parallel review subagents without them competing for your main conversation's context budget.

When I'm finishing up a doc, there are several things I want to check that don't depend on each other. Here are three saved subagents I've configured:

**Technical accuracy reviewer** (`.claude/agents/tech-reviewer.md`):

```markdown
---
name: tech-reviewer
description: Verifies documentation matches actual code behavior and internal sources of truth
tools: Read, Grep, Glob, Bash
---

You are a technical accuracy reviewer for documentation.

When invoked:
1. Read the documentation being reviewed
2. Find the corresponding source code in src/
3. Check internal docs in /docs/internal/ for canonical behavior
4. Verify every claim in the doc matches actual implementation

For each section, report:
- ✓ Accurate (code matches doc)
- ⚠ Needs attention (minor discrepancy)
- ✗ Incorrect (documentation contradicts code)

Include specific line numbers and suggest corrections.
```

**Code sample tester** (`.claude/agents/sample-tester.md`):

```markdown
---
name: sample-tester
description: Runs all code samples in documentation to verify they work
tools: Bash, Read, Write
---

You are a code sample tester for documentation.

When invoked:
1. Extract all code blocks from the documentation
2. Set up test environment: npm install or pip install as needed
3. Run each sample, capturing output and errors
4. Verify samples produce expected results

For each code sample, report:
- ✓ Works (runs without errors, output looks correct)
- ⚠ Runs with warnings (works but has non-critical issues)
- ✗ Fails (throws error or produces wrong output)

Include the exact error message and suggested fix.
```

**User confusion checker** (`.claude/agents/confusion-checker.md`):

```markdown
---
name: confusion-checker
description: Compares documentation against real user questions from GitHub issues
tools: Bash, Grep, Read
---

You are a user experience reviewer for documentation.

When invoked:
1. Search GitHub issues for user questions about this feature
2. Identify common points of confusion
3. Check if the documentation addresses these confusions
4. Flag gaps where users are still getting stuck

For each common question, report:
- ✓ Addressed (doc clearly explains this)
- ⚠ Partially addressed (mentioned but unclear)
- ✗ Missing (users ask this but doc doesn't cover it)

Suggest specific additions or clarifications.
```

I can kick off all three at once. They run in parallel. I keep working (or take a coffee break), and when they're done I get three focused reports back. No context-switching, no re-explaining the same instructions, no waiting for one review to finish before starting the next.

The progression was: prompting, then spawning subagents ad hoc, then saving subagent configurations. Each step removed a layer of repetition.

([Full subagents documentation](https://code.claude.com/docs/en/sub-agents))

### Hooks: Automation that runs without you asking

Hooks are scripts that run automatically when certain things happen. You set them up once and forget about them.

A great one for technical writing: run your linter before any edit saves.

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Edit|Write",
      "command": "vale --config=.vale.ini $FILE"
    }]
  }
}
```

Now Claude checks your style guide automatically before making changes. You catch issues immediately instead of in code review.

To be fair, Claude will often do this for you anyway if you add it to a skill, or even if you just do it manually a few times and Claude catches on that it should run the linter whenever it edits docs. But hooks make it deterministic. No relying on Claude to remember, no hoping it picks up the pattern. It just happens every time.

Another one that's been surprisingly useful: a sound notification when Claude finishes a message. I thought it would be annoying, but I tend to context switch while waiting for responses, and now I know when to come back without constantly checking.

```json
{
  "hooks": {
    "PostMessage": [{
      "matcher": "assistant",
      "command": "afplay /System/Library/Sounds/Glass.aiff"
    }]
  }
}
```

There are a lot of places you can add hooks beyond just these. You can hook into the start of a session, before context compaction, after Claude sends a message, on session stop. Basically anywhere you'd want to inject a check or an action, there's probably a hook point for it.

([Full hooks documentation](https://code.claude.com/docs/en/hooks))

### Plugins: Bundle it all up and share it

Once you've built up a collection of slash commands, skills, and saved subagents, you can package them as a plugin. This is basically a way to bundle your whole setup and share it with anyone else.

A quick note on sharing: if you store your commands, skills, and agents in your project's `.claude/` folder and commit them to git, your team automatically gets them when they clone the repo. That works great for project-specific setups.

But if you want to share across multiple projects or distribute your setup more broadly, you can create and host a plugin. For technical writing, this is huge. Instead of every new person on the team figuring out the same workflows from scratch, you can hand them a plugin with everything already configured: the style guide in `CLAUDE.md`, the `/review` and `/simplify` commands, the API docs skill, the saved subagents for tech review and code testing. They install it and they're immediately working the same way you are.

It also means you can version and iterate on your team's setup. Found a better way to structure the tech review subagent? Update the plugin. Everyone gets the improvement.

To browse and install plugins, just run `/plugin`. This connects you to plugin marketplaces, which are registries of plugins you can install. The default one is Anthropic's, but you can [add other community marketplaces](https://code.claude.com/docs/en/discover-plugins) if you want to explore more.

One I use all the time: `commit-commands`. It auto-drafts commit messages based on my changes. Way more useful than "update skills docs" but also not something I want to spend brainpower on.

And if you've built something useful, consider publishing it to a marketplace. The more people share what's working for them, the better the ecosystem gets.

([Full plugins documentation](https://code.claude.com/docs/en/plugins))

### Where to store all this

Most of these tools can live at different scopes depending on how widely you want to share them:

| Scope | Location | Who gets it |
| ----- | ----- | ----- |
| **Enterprise** | Admin-configured | Everyone in your organization |
| **Personal** | `~/.claude/` | Just you, across all projects |
| **Project** | `.claude/` in your repo | Anyone who clones the repo |
| **Plugin** | Bundled with a plugin | Anyone with the plugin installed |

If two things have the same name, the higher row wins: enterprise overrides personal, personal overrides project, project overrides plugin.

So if you build a `/review` command that works great for your docs workflow, you can keep it personal, commit it to your project for the team, or (if you're an enterprise admin) roll it out to everyone. Same goes for skills, agents, and hooks.

## How these pieces fit together

Here's how I'm starting to think about it:

| Tool | What it's for | When to use it | Example |
| ----- | ----- | ----- | ----- |
| **`CLAUDE.md`** | Universal context | Guidelines that apply to every task | Voice, formatting rules |
| **Slash commands** | Repeatable prompts | Any prompt you use more than twice | `/review`, `/simplify` |
| **Skills** | Specialized instructions | How to approach specific doc types | API docs skill, tutorial skill |
| **Subagents** | Parallel work | Reviews that can run in the background | Tech reviewer, code tester |
| **Hooks** | Automatic guardrails | Checks you always want to run | Linter, sound notification |
| **MCP** | External systems | Access to tools and data outside your codebase | Slack, GitHub |

The goal isn't to use all of these. It's to know they exist so you can reach for them when they'd help.

## Where to start (for real)

If you're not using Claude Code yet:

1. Install it: follow the [quickstart guide](https://code.claude.com/docs/en/quickstart)
2. Navigate to a docs repo: `cd your-docs && claude`
3. Run `/init`

The `/init` command analyzes your codebase and helps you set up a `CLAUDE.md`, slash commands, and subagents based on what Claude thinks would be useful for your project. It's a great way to bootstrap everything at once instead of building from scratch.

If you're already using it but mainly just prompting:

1. **Beef up your `CLAUDE.md`** if you haven't already. Put universal guidelines there, and consider breaking out specialized stuff into skills.
2. **Create one slash command** for your most common task (maybe `/review`).
3. **Try spawning a subagent** next time you're finishing a doc and need a parallel review. Once you find yourself repeating the same instructions, save it as a configuration you can reuse.

You don't need to set up everything at once. Add tools when you notice friction.

---

Three weeks ago, I thought I'd mastered Claude Code with just `CLAUDE.md` and some good prompts. Turns out that was just the beginning. The tools I've been discovering since then haven't made the work feel automated or detached. They've made it feel more intentional. Less time repeating myself, more time designing workflows that actually help.

I'm still early in this myself. Three weeks in, still learning, still finding things I didn't know existed. But that's kind of the point. You can get huge value from the basics, and then keep discovering more as you hit friction.

If you're using Claude Code for docs work, I'd love to hear what you're exploring (or what you're still avoiding because it seems complicated).
