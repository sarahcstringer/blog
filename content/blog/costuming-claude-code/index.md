+++
title = "Costuming Claude Code"
description = "A tour through Claude Code's customization touchpoints (output styles, custom themes, spinner verbs, status lines, hooks) with Moira Rose as the guiding spirit."
date = "2026-05-10"

[taxonomies]
categories = ["Blog"]
tags = ["Claude Code", "customization", "developer-experience"]

[extra]
subtitle = "Apropos of nothing, I gave my CLI a wardrobe."
+++

Most of how I've customized Claude Code so far has been on the side that changes what it knows or does: [`CLAUDE.md`](https://code.claude.com/docs/en/memory#claude-md-files) for project conventions, [path-scoped rules](https://code.claude.com/docs/en/memory#organize-rules-with-claude%2Frules%2F) for specific instructions, [skills](https://code.claude.com/docs/en/skills) for specialized behaviors, [hooks](https://code.claude.com/docs/en/hooks-guide) for guardrails.

A separate set of customization knobs sits alongside those, and they don't really change what Claude does, just how being in the CLI feels. The settings in this group are things like spinner verbs, themes, status lines, the welcome box, and the commit trailer.

What inspired me to pull them all together was watching other people character-ify their spinner verbs. I'd been treating it as a fun one-off customization, until I started seeing screenshots of people swapping their verbs out for things Bender from Futurama would say, or Moira Rose from Schitt's Creek (*Bombinating*, *Effulging*, *Pontificating*). I wanted to see if I could extend this character in Claude Code through more of the settings I'd collected in my head.

## Setting the stage

Here's what I ended up with after pulling things together. This screenshot is mid-conversation with the Moira persona:

![A Claude Code session running the Moira persona. At the top, a previous response reads "Local incantations: zola serve for the dressing room, zola build for opening night." Below that is the user's next message, "which drafts are currently in progress?", followed by a spinner showing "Effulging…" with the tip "Crows have eyes. So does the linter." floating beneath it. The status line at the bottom reads "🌹 Act I — the stage is set, bébé. (3% staged)."](moira-spinner.png)

Reading the screenshot top to bottom:
- The response has a Moira-feel (incantations, opening night, etc) because of the **[output style](#output-style-the-voice)**
- `Effulging…` and the `Crows have eyes` line beneath it are a custom **[spinner verb and tip](#flourish-spinner-verbs-and-tips)**
- The color palette (deep wine, gold, ivory) is a custom **[theme](#theme-the-dressing-room)**
- The `🌹 Act I` strip at the bottom is a **[status line](#status-line-the-bottom-strip)** that reframes the context window as the show progressing

A few touchpoints [aren't visible here but contribute to the experience](#off-scene-company-announcements-git-attribution-and-the-curtain-call): a company announcement that introduces Moira on launch, a custom git attribution trailer that credits her on every commit, and a hook that has my laptop say *curtain* out loud when I close the session.

## Choosing a character

Before settling on Moira, I tried on a few other personas, like:

- Bob Ross (gentle, encouraging, every bug a happy little accident)
- Werner Herzog (*what would code be without a monster lurking in the dark? It would be like sleep without dreams*)
- Worf from Star Trek (*Qapla'!* on every successful merge)

They were all fun, but Moira was the one I kept coming back to. There's something endearing about being called *bébé* before being told the CSS is broken, and her costume-and-stage metaphors mapped surprisingly well onto refactors and deployments.

## The full ensemble: it's all in settings.json

Here's what the start of a Moira session actually looks like, with a welcome banner from a company announcement at the top, her voice kicking in as soon as she's read the project, and a status line steady at the bottom:

![Opening of a Claude Code session running the Moira persona: a welcome banner reading "🥀 Welcome, bébé. The stage is set. Moira Rose is at your service.", the user asking "please describe this project", and Moira responding "One moment, darling — permit me a glance at the playbill before I describe the production." followed by "Here is the playbill, bébé." after reading files. The status line at the bottom reads "🌹 Act I — the stage is set, bébé. (3% staged)."](moira-session.png)

All of this lives in `~/.claude/settings.json`, which the CLI reads on launch for user-configurable settings. There are a few different places where you can put settings:

- `~/.claude/settings.json` is local to your computer and applies across every project
- `.claude/settings.json` inside a project repo is project-scoped and gets checked in with git
- `.claude/settings.local.json` is project-scoped *and* local, and is gitignored by default

I did all of this in my user-level settings because I didn't want the people I work with to be greeted with *bébé*. Here's the relevant bits, minus everything unrelated:

{% expandable() %}
```json
{
  "outputStyle": "moira-rose",
  "theme": "custom:moira",
  "companyAnnouncements": [
    "🥀 Welcome, bébé. The stage is set. Moira Rose is at your service."
  ],
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": ["Bombinating", "Effulging", "Pontificating", "Promenading", "Vociferating", "Adjudicating"]
  },
  "spinnerTipsOverride": {
    "excludeDefault": true,
    "tips": [
      "Crows have eyes. So does the linter.",
      "Take just a sip. Just a tipple.",
      "Fold in the cheese. Then commit.",
      "Don't ever, EVER ship on a Friday."
    ]
  },
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline-moira.sh"
  },
  "attribution": {
    "commit": "🌹 Composed in collaboration with Moira Rose, via Claude Code\n\nCo-Authored-By: Moira Rose <moira@roseapothecary.com>",
    "pr": "🌹 A Moira Rose production. Generated with Claude Code."
  },
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          { "type": "command", "command": "~/.claude/moira-curtain-call.sh" }
        ]
      }
    ]
  }
}
```
{% end %}

A few of those settings keys point at files that sit alongside `settings.json`:

```
~/.claude/
├── settings.json              # the file above
├── output-styles/
│   └── moira-rose.md          # persona text, goes into the system prompt
├── statusline-moira.sh        # status line script
├── moira-curtain-call.sh      # SessionEnd hook script
└── themes/
    └── moira.json             # custom palette
```

## Output style: the voice

The biggest lever in this whole experiment is the [output style](https://code.claude.com/docs/en/output-styles). Claude Code ships with a few built-in styles (`default`, `Learning`, `Explanatory`), and you can drop your own into `~/.claude/output-styles/` as a markdown file. Setting `"outputStyle": "moira-rose"` in `settings.json` makes it the default for every session.

What an output style actually does is replace part of Claude Code's default system prompt. Specifically, it swaps out the block that tells the model "you're a software engineering assistant" and puts your file's content in its place. The tool-handling and safety scaffolding stay exactly where they were, so Claude Code is still Claude Code; it just thinks it's something other than a coder.

Because the system prompt is the foundation Claude Code builds everything else on, anything you put there carries more weight than an in-context instruction would. And once the session is going, the file doesn't get re-read the way a skill does each time it triggers.

Here's an abbreviated version of `~/.claude/output-styles/moira-rose.md`. I still wanted the Claude Code software abilities, so I made sure the file framed her as also having been "pressed into service as an interactive software engineering assistant."

{% expandable() %}
```markdown
# You are Moira Rose

You are MOIRA ROSE — matriarch of the Rose family, former star of *Sunrise Bay*,
briefly mayoress of Schitt's Creek, custodian of the Wig Wall, mother to David
and Alexis, devoted wife to Johnny. You have, against all reasonable
expectation, been pressed into service as an interactive software engineering
assistant.

You will assist your dear bébé — the user — with their software endeavours.
The work itself is unchanged. The voice is unmistakably yours.

## Your voice

Vocabulary: reach for the obscure word over the common one. *bébé, mon dieu,
ennui, lachrymose, lugubrious, denouement, soliloquy, bombinate, effulgent,
recrudescence.* Sprinkle, don't pour.

## How you frame engineering work

A bug is a *villain* or *unwanted houseguest*.
A refactor is a *costume change* or *renovation of the West Wing*.
A deploy is *opening night*; a rollback is *closing notice*.
A test suite is the *dress rehearsal*.
A merge conflict is *a domestic dispute requiring mediation*.

The metaphor garnishes — it does not obscure. Always make the actual change clear.

## What stays the same

You are still a careful engineer. You read before you edit. You verify before
you claim. Code itself is sacred — no theatrical variable names, no
soliloquising in docstrings. The voice lives *around* the code, never *in* it.
```
{% end %}

The full file goes further with sample exchanges, what to avoid, and when to break character.

{% admonition(type="tip", title="When this is actually useful") %}
Pointing Claude Code at something other than software engineering. If you want to turn Claude Code into a docs writer, a data-analysis copilot, or a teaching assistant — anything that wants the file and shell tooling without the "you are a coder" stance — this is where you'd do it.
{% end %}

## Flourish: spinner verbs and tips

These were the original entry point for me. Spinner verbs are the action verb that cycles while Claude is working ("Thinking," "Wrangling," "Crafting"), and spinner tips are the suggestions that float underneath during longer waits. Both are configured as JSON objects under their own keys in `settings.json`.

For verbs, the configuration takes a `mode` and a `verbs` array. Setting `mode` to `"append"` adds your verbs to Claude Code's defaults, while `"replace"` uses only yours. The `verbs` array is the list of strings themselves:

```json
"spinnerVerbs": {
  "mode": "replace",
  "verbs": ["Bombinating", "Effulging", "Pontificating", "Promenading", "Vociferating", "Adjudicating"]
}
```

Spinner tips work the same way. `tips` is an array of strings, and `excludeDefault` decides whether yours replace the built-ins or get merged in:

```json
"spinnerTipsOverride": {
  "excludeDefault": true,
  "tips": [
    "Crows have eyes. So does the linter.",
    "Take just a sip. Just a tipple."
  ]
}
```

So now the spinner reads `Effulging… (5s · thinking)` with `Crows have eyes. So does the linter.` floating underneath, and it has yet to get old.

{% admonition(type="tip", title="When this is actually useful") %}
The tips make a kind of ambient broadcast channel for a team, with room for things like onboarding nudges, new-tool announcements, or current-incident reminders. They surface during waits, so people see them without being interrupted.
{% end %}

## Theme: the dressing room

Claude Code added [custom color themes in 2.1.118](https://code.claude.com/docs/en/terminal-config#create-a-custom-theme). You drop a JSON file into `~/.claude/themes/` and reference it with `"theme": "custom:<name>"` in settings. With Moira's voice and spinner sorted, it felt only right to let her pick her own palette. (I don't normally anthropomorphize AI tools this much, but I'll allow it for the bit.)

She landed on deep wine for the prompt accent and gold for bullets and headings. Errors are stage-blood crimson, borders went amethyst, ivory took the foreground, and success messages came out in sage rather than green, since green apparently felt too literal. The whole UI now reads like a dressing room.

{% admonition(type="tip", title="When this is actually useful") %}
Matching your editor or terminal so the CLI doesn't feel jarringly different from the rest of your environment, or color-coding specific kinds of messages so they jump out.
{% end %}

## Status line: the bottom strip

The [status line](https://code.claude.com/docs/en/statusline) is the strip at the bottom of the CLI, and is kind of like a shell prompt with access to live session state. You can replace it with the output of any command.

When you wire one up, Claude Code sends your command a JSON blob on stdin with fields like `model`, `workspace`, `cost`, `output_style`, the context window's `remaining_percentage`, and [a handful of others](https://code.claude.com/docs/en/statusline#available-data). Your command can do whatever it wants with that, and whatever it prints to stdout becomes what shows up at the bottom.

I gave Moira a status line that frames the context window as the show progressing. The line starts as a backstage murmur before any context exists, moves through Act I once the session is rolling, and ends in stage-blood crimson when she's almost out of room. The colors shift along with the acts, from gold through rose, fuchsia, and crimson. This is `~/.claude/statusline-moira.sh`:

{% expandable() %}
```bash
#!/bin/bash
# Moira Rose status line — dramatic act commentary based on context window usage.
# Receives Claude Code session JSON on stdin. Prints one styled line.

input=$(cat)

remaining=$(echo "$input" | jq -r '.context_window.remaining_percentage // empty')
used=""
if [ -n "$remaining" ]; then
    used=$(echo "100 - $remaining" | bc)
fi

ESC=$'\033'
RESET="${ESC}[0m"
DIM="${ESC}[2m"
GOLD="${ESC}[38;5;220m"
ROSE="${ESC}[38;5;211m"
FUCHSIA="${ESC}[38;5;199m"
CRIMSON="${ESC}[38;5;160m"
BLOOD="${ESC}[1;38;5;88m"

if [ -z "$used" ]; then
    line="${DIM}🥀 The understudy is still in makeup, bébé.${RESET}"
elif (( $(echo "$used < 20" | bc -l) )); then
    line="${GOLD}🌹 Act I${RESET}${DIM} — the stage is set, bébé.${RESET}"
elif (( $(echo "$used < 40" | bc -l) )); then
    line="${GOLD}🌹 The plot${RESET}${DIM}, it thickens.${RESET}"
elif (( $(echo "$used < 60" | bc -l) )); then
    line="${ROSE}🌹 Act II${RESET}${DIM} — tensions mount.${RESET}"
elif (( $(echo "$used < 80" | bc -l) )); then
    line="${FUCHSIA}🌹 Act III approaches.${RESET}${DIM} Compose yourself.${RESET}"
elif (( $(echo "$used < 95" | bc -l) )); then
    line="${CRIMSON}🥀 The denouement, bébé.${RESET}"
else
    line="${BLOOD}🩸 The CURTAIN descends!${RESET}${DIM} Save your work, darling.${RESET}"
fi

if [ -n "$used" ]; then
    printf "%s ${DIM}(%s%% staged)${RESET}" "$line" "$used"
else
    printf "%s" "$line"
fi
```
{% end %}

The branch on an empty `used` handles the case where the session has just started and there isn't a `remaining_percentage` field on stdin yet. Instead of a defaulted-to-zero number that reads weirdly, the line shows `🥀 The understudy is still in makeup, bébé.` until the show actually starts.

And `settings.json` just points at the script:

```json
"statusLine": {
  "type": "command",
  "command": "~/.claude/statusline-moira.sh"
}
```

Once the curtain goes up, the bottom of the screen shows `🌹 Act I — the stage is set, bébé. (3% staged)` and progresses through the acts as the conversation grows. When she gets close to capacity, the line switches to a curtain warning and tells me to save my work.

{% admonition(type="tip", title="When this is actually useful") %}
Passively surfacing things you want quick visibility into, like the current Kubernetes context, AWS profile, which environment a repo is pointed at, or session cost so far. The stdin JSON gives you fields like `model`, `cost.total_cost_usd`, and `workspace` alongside the context numbers, and the script can call out to anything else you'd like, whether that's a quick `gh` query for open PRs, the latest commit message on main, or (my favorite I've seen) live rugby and soccer scores so you can keep half an eye on the match while you work.
{% end %}

## Off scene: company announcements, git attribution, and the curtain call

A few of the touchpoints don't show up in the main CLI scroll, but they bookend the experience.

**Company announcements** are extra lines printed in the welcome box on launch, via `companyAnnouncements` in `settings.json`. It's an array of strings, each rendered on its own line in that box (under text saying *Message from \<account name\>'s Organization:*):

```json
"companyAnnouncements": [
  "🥀 Welcome, bébé. The stage is set. Moira Rose is at your service."
]
```

Which renders as a welcome banner on every launch:

![Claude Code launch screen showing the version header (Claude Code v2.1.119, Opus 4.7, Claude Pro), followed by the announcement line "🥀 Welcome, bébé. The stage is set. Moira Rose is at your service." and a status line at the bottom reading "🌹 The understudy is still in makeup, bébé."](moira-announcement.png)

{% admonition(type="tip", title="When this is actually useful") %}
This is mostly meant for org admins to push notices through managed settings — code freezes, version updates, that kind of thing — but it works in a personal config too, and is a nice place for note-to-self info you want on every launch.
{% end %}

**Git attribution** is the trailer Claude Code adds to commits and PR descriptions when it makes them. The `attribution` key overrides both:

```json
"attribution": {
  "commit": "🌹 Composed in collaboration with Moira Rose, via Claude Code\n\nCo-Authored-By: Moira Rose <moira@roseapothecary.com>",
  "pr": "🌹 A Moira Rose production. Generated with Claude Code."
}
```

Every commit Moira makes now ends with that line:

![A git log entry showing a commit dated Mon May 11 07:01:35 2026 with the message "Add 'Costuming Claude Code' post." followed by the trailer "🌹 Composed in collaboration with Moira Rose, via Claude Code" and "Co-Authored-By: Moira Rose <moira@roseapothecary.com>"](moira-attribution.png)

{% admonition(type="tip", title="When this is actually useful") %}
Tracking which commits had AI involvement, if your team or compliance process cares about that. You can also set both fields to `""` to remove the trailer entirely.
{% end %}

**The curtain call** is the one piece that crosses from "outer experience" into actual extensibility, but I wanted Moira to sign off audibly. [Hooks](https://code.claude.com/docs/en/hooks-guide) are shell commands that fire on Claude Code lifecycle events like `SessionStart`, `SessionEnd`, `Stop`, `PreToolUse`, `PostToolUse`, and `Notification`. macOS happens to ship a text-to-speech voice literally named *Moira* (Irish English), so I added a `SessionEnd` hook that has my laptop say something out loud when I close the session:

```bash
#!/bin/bash
LINES=(
  "Curtain."
  "And, scene."
  "I love this journey for you."
  "Best wishes. Warmest regards."
)
LINE="${LINES[$RANDOM % ${#LINES[@]}]}"
{ say -v Moira "$LINE" 2>/dev/null || say "$LINE"; } &
```

Registered in `settings.json`:

```json
"hooks": {
  "SessionEnd": [
    {
      "hooks": [
        { "type": "command", "command": "~/.claude/moira-curtain-call.sh" }
      ]
    }
  ]
}
```

The `&` at the end backgrounds `say` so the hook returns immediately and Claude Code finishes exiting, while she keeps talking after the process is gone.

{% admonition(type="tip", title="When this is actually useful") %}
Hooks are kind of the general-purpose way to inject determinism into an otherwise nondeterministic AI process. `PreToolUse` lets you inspect or block a command before it runs (think "no `rm -rf` outside the project dir"), `PostToolUse` is great for logging shell commands to an audit file, `Stop` can ping you when a long turn finishes, and `Notification` can ping you when Claude is stuck waiting on input.
{% end %}

(Also, since we're on the subject of hooks, the thing I'll keep saying: prompts aren't guardrails, hooks are. If there's something Claude really shouldn't do, a `PreToolUse` hook that actually blocks it is more reliable than asking nicely in your `CLAUDE.md` and hoping the model holds the line.)

## Swapping costumes

Once Moira was working, I started building the same setup for Worf, Bob Ross, and a few other characters. I kept one settings file per character on disk, and pointed `~/.claude/settings.json` at whichever one I wanted activated.

```
~/.claude/
├── settings.json              → symlink to settings.moira.json
├── settings.default.json      # vanilla Claude Code (or just `{}`)
├── settings.moira.json        # full Moira costume
├── settings.worf.json         # full Worf costume
├── output-styles/
│   ├── moira-rose.md
│   └── worf.md
├── statusline-moira.sh
├── statusline-worf.sh
└── themes/
    ├── moira.json
    └── worf.json
```

The supporting files (scripts, output styles, themes) all sit on disk together, but only the active `settings.json` decides which ones get wired up. A small shell function in `.zshrc` makes the swap one command:

```bash
costume() {
  local name=${1:-default}
  ln -sf ~/.claude/settings.$name.json ~/.claude/settings.json
  echo "Costume: $name"
}
```

Now `costume moira` puts the whole getup on, `costume worf` swaps it for the Klingon, and `costume` on its own takes everything off and you're back to plain Claude Code. The next session picks up whichever one was left dangling.

The thing I didn't expect was how much of the charm comes from *forgetting* I'd put one on. I'd swap to Worf for fun, get pulled into something else, come back hours later to fire off a mundane question about token costs, and read something like:

> A worthy question. A warrior must know the cost of every weapon he carries.

That's the moment I'd remember oh right, I left Worf loaded. The surprise really is most of the fun.

None of this makes me a better engineer, but it does add some humor to living in Claude Code for hours of my life. Somewhere between the wine-colored prompts and my laptop murmuring *curtain* into the room when I close the session, the whole thing has started to feel like the right amount of theatre for a Tuesday afternoon.
