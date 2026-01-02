+++
title = "Claude Code beyond prompting"
description = "Turns out I'd been using a Swiss Army knife with only the main blade"
date = "2025-12-31"

[taxonomies]
categories = ["Blog"]
tags = ["AI", "Claude Code", "technical writing", "tooling", "workflows"]
+++

Before I joined Anthropic, I thought I was pretty good at Claude Code.

I was using it to speed up my technical writing: generating diagrams I couldn’t make myself, catching inconsistencies quickly, and automating the most tedious parts of doc review. Putting our entire style guide into a `CLAUDE.md` made it even better. It wasn’t perfect, but it removed whole classes of style-related mistakes and shifted every edit closer to “done.”

That felt like a complete workflow. I’d heard about things like subagents, skills, and hooks, but what I had was working well, so I never looked much deeper.

On my first day writing documentation for Claude Code, I realized I’d been using a Swiss Army knife with only the main blade. The other tools were there the whole time, each better suited for certain kinds of work. I just hadn’t learned when to reach for them.

That realization is what led me to start forming a clearer mental model for how these tools fit together.

*Note: this is a personal mental model I’ve been building through hands-on use, not an official or exhaustive guide.*

## Where I started (and where you might be too)

The `CLAUDE.md` file was my first big unlock. If you haven’t done this yet, start here: create a `CLAUDE.md` file in your project root (or in a `.claude/` directory) and put your universal writing guidelines there. It’s the highest-impact, lowest-effort improvement you can make.

At first, I put everything in it. This included documentation philosophy, voice and tone, grammar rules, formatting conventions, and notes about our publishing workflow. Because Claude reads this file automatically at the start of every conversation, every edit followed our conventions without repeated reminders.

That worked well. The issue was not usefulness, but scope. I started adding highly specific, task-level instructions and rewriting the same guidance. Since `CLAUDE.md` loads in full at the start of every conversation, every extra byte counts toward your context window and token usage. It’s best suited to universal rules, not specialized knowledge you only need occasionally.

## What else Claude Code can do (that I wasn’t using)

These are the tools you can reach for beyond prompting and `CLAUDE.md`, and the kinds of technical writing tasks each one is best suited for.

**Quick summary:**
- [Slash commands](#slash-commands-one-prompt-saved-forever): saved prompts you invoke by name
- [Skills](#skills-context-that-loads-only-when-you-need-it): specialized instructions that load on demand
- [Subagents](#subagents-parallel-actions-without-losing-focus): parallel work with isolated context
- [Hooks](#hooks-automation-that-runs-without-you-asking): automatic checks and actions
- [Plugins](#plugins-bundle-it-all-up-and-share-it): bundle everything and share it

You don’t need to adopt all of these at once. Start with `CLAUDE.md`, then add the others only when you feel friction.

### Slash commands: One prompt, saved forever

Start with [the built-in slash commands](https://code.claude.com/docs/en/slash-commands): `/rename` to clean up session names, `/init` to get a project off the ground.

Beyond the built-ins, any prompt you type more than twice can be a custom command. They're Markdown files you store in `.claude/commands/` and trigger within a Claude session.

Here are two examples you might add to a technical writing toolkit:

**`/review`** (`.claude/commands/review.md`), a full doc review checklist:

```markdown
Review this document for:
1. Style guide compliance (check CLAUDE.md)
2. Technical accuracy (do the code samples work?)
3. Broken links and references
4. Missing context (would a new user understand this?)
5. Structural issues (headers, flow, completeness)

Categorize issues as: blocking, should fix, nice to have.
```

**`/simplify`** (`.claude/commands/simplify.md`), for when you've written something too dense:

```markdown
Simplify this content for a developer who's new to the product.
- Remove jargon or define it on first use
- Shorten sentences
- Cut anything not essential to the core concept
- Add an example if it would help
```

You can run `/review` or `/simplify` and Claude will run that prompt on the content you've been working on.

([Full slash commands documentation](https://code.claude.com/docs/en/slash-commands))

### Skills: Context that loads only when you need it

I was also using `CLAUDE.md` for specialized workflows like API doc structure and tutorial formats. But `CLAUDE.md` eats into your token budget from the start of every conversation and not all of the information might be relevant to every session. Skills solve this.

They're directories with instructions, templates, and reference files. At startup, Claude only loads skill names and descriptions. The full `SKILL.md` file only loads when Claude decides from the task to use a skill.

In technical writing, one skill use case might be directions for different doc types, like a skill for API docs vs. tutorials. An API docs skill might look like this:

**API documentation skill:**

```
.claude/skills/api-docs/
├── SKILL.md           # Instructions for API doc structure
├── template.md        # Standard template with all sections
└── examples/
    ├── good-example.md    # "Here's what we're going for"
    └── common-mistakes.md # "Avoid these patterns"
```

The `SKILL.md` file for this skill would look like:

````markdown
---
name: documenting-apis
description: Provides structure and examples for writing API reference documentation. Use when creating or editing API documentation.
---

# API Documentation

## Structure
Every API doc needs: Overview, Authentication, Endpoints, Error codes, Examples

## Voice
More formal than tutorials. Assume reader knows HTTP basics.

## Template
See [template.md](template.md) for the standard structure. Don't skip sections.

## Examples
See [examples/good-example.md](examples/good-example.md) for reference patterns.
````

The supporting files (templates, examples) only load when the task actually needs them. Claude discovers them through links in your `SKILL.md`, so you can keep detailed reference material available without it eating up context until it's relevant.

Now your `CLAUDE.md` can stay lean with only universal guidelines for your project, while specialized context loads automatically when you need it.

([Full skills documentation](https://code.claude.com/docs/en/skills))

### Subagents: Parallel actions without losing focus

Subagents are separate Claude sessions that run in the background with their own context. The main agent passes specific context to them, and they report back when done.

You can spawn them ad hoc for one-off tasks. But you can also save subagent configurations as Markdown files with YAML frontmatter in `.claude/agents/`. Once configured, they become reusable tools you invoke by name.

I frequently use saved subagents for the end-of-doc review process: tech review, UX review, style review. They run in isolated context, which means I can kick off multiple reviews in parallel without competing for the main conversation's token budget.

Here's an example of a saved subagents you might configure for this type of work (and you can use the `/agents` slash command to help you build these out instead of doing it manually):

**Technical accuracy reviewer** (`.claude/agents/tech-reviewer.md`):

```markdown
---
name: tech-reviewer
description: Verifies documentation matches actual code behavior and internal sources of truth. Use when reviewing documentation for technical accuracy or when code has changed.
tools: [Read, Grep, Glob, Bash]
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

When you're ready for a review, you can kick this off and can run it in parallel with other subagents you have. You keep working (or take a coffee break), and when the subagent is done, you get a focused report back in the main conversation. 

([Full subagents documentation](https://code.claude.com/docs/en/sub-agents))

---

#### Subagents vs. skills vs. slash commands

You might start to notice some overlap between these tools. Could the `/review` slash command also be a subagent? Or a skill? Yes; in some cases, you might even want a slash command and a subagent that perform similar tasks. The difference is not what they do, but how they handle context and who triggers them.

| Tool | Context/isolation | Usually triggered by | When to use it |
| ----- | ----- | ----- | ----- |
| **Slash command** (`/review`) | Full main conversation context | You (explicit invocation: `/review`) | When you want Claude to consider the entire discussion and current project state. |
| **Subagent** (`review-agent`) | Isolated context (only what you/agent pass to it) | You (tell Claude to run it) | When you want to run tasks in parallel or keep your main conversation focused. |
| **Skill** (`review-skill`) | Loads specialized instructions on demand | Claude (infers from task context) | When you need structure, templates, or reference material without keeping it in context all the time. |

*Note: The triggering can get messier in practice; for example, Claude can invoke custom slash commands, and you can explicitly ask Claude to use a specific skill. The table above covers the most common usage.*

You can use more than one of these for the same kind of task. A slash command is useful for quick, context-aware checks. A subagent is better for parallel or background work. A skill is the right choice when the work needs deeper, reusable guidance without bloating your main context.

---

### Hooks: Automation that runs without you asking

Hooks are scripts that run automatically when certain Claude Code events happen. You set them up once and then forget about them.

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

Now whenever Claude edits your work, it runs linting automatically. You catch issues immediately instead of in code review.

Hooks make checks deterministic. No relying on Claude to remember, no hoping it picks up the pattern. It just happens every time.

Another useful one: a sound notification when Claude finishes a message. I tend to context switch while waiting for responses, and this tells me when to come back without constantly checking.

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

There are a lot of places you can add hooks beyond just these. You can hook into the start of a session, before context compaction, after Claude sends a message, on session stop; anywhere you'd want to inject a check or an action, there's probably a hook point for it.

([Full hooks documentation](https://code.claude.com/docs/en/hooks))

### Plugins: Bundle it all up and share it

Once you've built up a collection of slash commands, skills, subagents, and hooks, you can package them as a [plugin](https://code.claude.com/docs/en/plugins). This is a way to bundle your whole setup and then you can share it with others via a [marketplace](https://code.claude.com/docs/en/plugin-marketplaces).

A quick note on sharing: if you store your commands, skills, and agents in your project's `.claude/` folder and commit them to git, your team automatically gets them when they clone the repo. That works great for project-specific setups.

If you want to share across multiple projects or distribute your setup more broadly, you can create and host a plugin. This can be great for cross-functional stakeholders who also do technical writing, for example; instead of every person figuring out the same workflows from scratch, you can hand them a plugin with everything already configured.

To browse and install prebuilt plugins, run `/plugin`. This connects you to plugin marketplaces, which are registries of plugins you can install. The default marketplace from Anthropic is automatically included, but you can [add other community marketplaces](https://code.claude.com/docs/en/discover-plugins) if you want to explore more.

One plugin I use all the time is `commit-commands` from the Anthropic marketplace. It auto-drafts commit messages based on my changes. Way more useful than "update skills docs" but also not something I want to spend brainpower on.

To get it:
- Run `/plugin` within Claude Code
- Search for "commit-commands"
- Install it via the interactive menu

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

If you’re not using Claude Code yet:

1. Install it by following the [quickstart guide](https://code.claude.com/docs/en/quickstart).
2. Navigate to a docs repo and open Claude Code: `cd your-docs && claude`.
3. Start chatting

If you're already using it but mainly just prompting:

1. **Run `/init`**: analyzes your codebase and sets up `CLAUDE.md`, slash commands, and subagents based on what would be useful for your project.
2. **Add universal guidelines to `CLAUDE.md`**: voice, formatting rules, repo structure.
3. **Create one slash command** for your most common task (maybe `/review`).
4. **Spawn a subagent** for parallel reviews. When you repeat the same instructions, save it as a configuration.

You don't need to set up everything at once. Add tools when you notice friction.
