+++
title = "Reader as actor"
description = "My newest style nit: making sure the reader, not the file or the setting or the config, is the actor in the sentence."
date = "2026-08-13"

[taxonomies]
categories = ["Blog"]
tags = ["technical writing", "style", "editing", "AI"]

[extra]
subtitle = "My newest style nit obsession"
+++

Last week, I wrote the words "reader as actor" about 200 times in PR reviews. I updated a few skills and reworked lots of sentences around this theme. Making sure the reader is the actor in the docs is my newest obsession, the latest sentence-level nit I've started noticing everywhere.

To me, the phrase "reader as actor" means that the reader should be the main character in the docs' narrative. The reader ultimately needs to take what the docs tell them and use that effectively. When we elevate a different aspect of the interface to the actor in the narrative (ex: "the settings file lives in the root folder" – the settings file is now the main character in this sentence), we're obscuring what we actually want the user to do, know, or observe. If I'm reading the docs and see "the settings file lives in the root folder," does that mean you want me to put it there? Or it's already there and I edit it? Or the program just does it without me doing anything?

It's not that the reader isn't smart enough to put two and two together, but that they shouldn't need to. They're already piecing together how a complicated piece of software works, often when something's actively broken and they want to figure out what's wrong, so I want to take away the extra mental burden of deciphering the indirect language and just say what needs to happen.

With AI added to the mix, making sure the reader's actions are clearly stated helps close gaps where AI would otherwise infer. Where a reader might take an ambiguous statement and be confused and read around for more context, AI will sometimes insert a confidently incorrect conclusion.

Now that I'm on the lookout for this pattern, I notice it all over the place, across docs sites. Here are a few examples I pulled out of existing docs when skimming for examples of what I would call out in a review:

> Workspace or folder specific tasks are configured from the tasks.json file in the .vscode folder for a workspace.

<p class="fm-rewrite">→ Possible rewrite: To configure workspace- or folder-specific tasks, edit the <code>tasks.json</code> file in your workspace's <code>.vscode</code> folder.</p>

> When working locally in a project, a .npmrc file in the root of the project (ie, a sibling of node_modules and package.json) will set config values specific to this project.

<p class="fm-rewrite">→ Possible rewrite: To set config values for a single project, add a <code>.npmrc</code> file to the project root, next to <code>node_modules</code> and <code>package.json</code>.</p>

> A configuration can only provide one backend block.

<p class="fm-rewrite">→ Possible rewrite: Include only one backend block per configuration.</p>

With each one of these, I can piece together the actual intention and required actions from the surrounding context, but I'd argue it's an unnecessary indirection that deserves a rewrite. All of these could be clearer by centering the reader's actions and not making a file, folder, configuration, etc, the main character in the sentence.

A lot of existing style rules circle around this rule as well, like active over passive voice. But the reader as actor framework helps me quickly articulate what feels off in a sentence more than those other pieces of guidance have. Even if the sentence has other weird issues, rewriting the sentence to focus on what the reader needs to do or observe often smoothes the rest out and makes the prose noticeably better on the next pass. It gives me a vocabulary when I previously would have said "this feels awkward".

I don't expect "reader as actor" to catch everything. Passive voice and vague referents will keep sneaking past it, along with plenty of other issues I don't have a tidy label for yet. But it's given me a way to name what's off before I can always explain why, which is more useful to me right now than another rule to check against.

---

**Note 1**: This isn't an absolute. Models, programs, and other things can be actors when they legitimately drive, decide, act. A setting can turn something on or off (but really, the reader configures the setting, which then does something). Changelogs often read better with more abstract concepts as the actor.

---

**Note 2**: This all finally clicked for me after I read [*Style: Lessons in Clarity and Grace*](https://www.amazon.com/dp/0134080416) by Joseph Williams. It's a great book, and it starts from a similar premise, just more general: every sentence has an actor, which is whoever or whatever is doing the acting, and readers instinctively look for who that is. Williams leaves that actor open, but for my technical writing specifically, I've decided I always want that actor to be the reader, unless I have a real reason for it not to be.
