+++
title = "The git commands I avoided for nine years"
description = "A written recap of my Write the Docs talk about three git commands I avoided for most of my career, and the one I'd hand to past-me if I could only pick one."
date = "2026-05-15"

[taxonomies]
categories = ["Blog"]
tags = ["git", "talks", "developer-experience"]

[extra]
subtitle = "Reflog reflog reflog (and two others)"
+++

<!--
TODO before publishing:
- copy videos from ~/src/git-talk-slides/videos/ into this directory:
  with-worktrees.mp4, with-reflog.mp4, update-refs.mp4
-->

I recently gave a [talk](https://www.youtube.com/watch?v=FiGT3XYICSE) at [Write the Docs](https://www.writethedocs.org/) about three git commands I'd been avoiding for most of my career, or in two cases didn't know existed at all. For the demo environment behind the talk, I built [All Caps as a Service](../making-acaas/), a fake API and docs site that turned into its own side quest.

To properly set expectations up front: I'm not a git expert. I've used git almost daily for a decade with a workflow I'll describe in a minute, and recently discovered some parts of git that changed my thinking on it. I gave the talk and wrote this post because you don't need to be a git expert to get a long way with the tool, or to look into the parts you've been avoiding.

<iframe
  width="100%"
  height="415"
  src="https://www.youtube.com/embed/FiGT3XYICSE"
  title="The git commands I avoided for nine years"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  allowfullscreen
></iframe>

![Sketchnote of the talk by Dennis Dawson. A timeline of sticky notes across the top reads "The Git Commands I Avoided for Nine Years (and why I wish I hadn't)." Below, three commands are illustrated: git worktree, described as creating a second checkout of the repo; git reflog, described as an undo button for recovering lost commits, deleted branches, and bad rebases; and git rebase --update-refs, described as automagically force-updating all local branches that point to commits being rebased. Jujutsu (jj) appears at the bottom, described as reimagining what a commit is with no staging area and conflicts as first-class objects. On the right, "commit often" is written vertically in large pink letters.](git-talk-sketchnote.jpg)

*Sketchnote by [Dennis Dawson](https://dennissdawson.wixsite.com/mr--dawson/portfolio), via the [Write the Docs Flickr](https://www.flickr.com/photos/writethedocs/55266348341/in/album-72177720333614185). I love the reflog drawing.*

## How I got here

For the majority of my time in tech, I used about six git commands: `add`, `commit -m "wip"`, `rebase -i`, `commit --amend`, `cherry-pick`, `push`. Add, save, clean up the history, polish, and push. It worked, and it produced nice clean PRs.

When I first learned git in a software engineering bootcamp, the framing was unambiguous. Git was esssential but could be dangerous. You could lose work, and you should stay inside a small set of safe commands. Above all, don't touch rebase while you're learning. If you got into a mess with it, you were on your own.

I eventually went to work on teams with other engineers, learned about collaborating on projects using git, added `git rebase -i` to my toolkit, and felt pretty accomplished. My daily git commands worked for the most part... except for the few times when I'd do a bad rebase, and think I lost hours of work, despair and start over. Or get into a really tangled merge conflict and just end up deleting the branch and pulling down again from the remote. I thought that was the way it was; git was powerful with some rough edges.

What changed this for me had nothing to do with git initially. Last year, I wanted to run two Claude Code sessions on the same repo on different branches at once, purely for parallelism. I didn't know how to do this, because as far as I knew, you could only have one branch checked out for a repo at a given time on your computer.

[The fix turned out to be `git worktree`](https://code.claude.com/docs/en/worktrees), a git feature rather than anything to do with Claude Code. It also addressed a paper cut I'd been living with for years that I didn't even realized I'd had: the stash-checkout-checkout-back-stash-pop dance every time I wanted to spin up a quick branch for a typo fix when I was in the middle of other work. Fixing a problem I hadn't let rise to consciousness made me wonder what else was there and what I should incorporate into my toolkit.

## The three commands

I picked these three commands because they were the most directly useful ones that I found, and, in the case of git reflog, changed my view of git entirely.

### `git worktree`

You're mid-edit on a quickstart change, and someone asks for a one-line typo fix on a different branch. The dance is `git stash`, `git checkout main`, `git checkout -b typo-fix`, fix it, `git checkout quickstart`, `git stash pop`. So much context switching and heaven forbid you get pulled away in the middle of it and have to remember where you were in the dance when you left off.

Worktrees solve this context switching problem. The command `git worktree add ../typo-fix main -b typo-fix` makes a second working directory on your machine, pointing at the same repo, on a new branch. Then you can cd into that directory (`../typo-fix`) and have two branches open at the same time, without needing to wind down/stash your work from one task to switch to the other.

The worktrees share the same `.git` history, but all the files are duplicated and won't clobber each other as you make changes in each branch. The quickstart edits stay sitting in the original directory, untouched. Open the new folder in a second editor window, fix the typo, push, and the original is exactly where you left it.

**When you'd reach for it:** any time you'd otherwise stash or do the checkout dance. Especially worth it if you context-switch between branches a lot.

**When it shipped:** 2015, a year before I started learning git.

### `git reflog`

A pronunciation note: it's *ref-log*, short for "reference log." I'd always pronounced it `git re-flog`, which made it sound made up and was a primary reason I didn't investigate it any further. Based on the people who came up to me after the talk, I'm not the only one who did this. This is also the one command that made me rethink my whole vision of git and realize it's so much safer than I'd been taught.

`git reflog` shows everywhere `HEAD` has been recently (every commit, reset, checkout, rebase, all of it). If you ever accidentally `git reset --hard` too far or drop a commit during a rebase, or get a tangled git history through a merge conflict, this is the command that saves you. The "lost" commit, or the good place you used to be but now aren't, is sitting right there in the reference log. You can always go back to it. Just run `git reflog`, find the SHA for the commit/place in time you want to return to, and `git reset` back to that place.

The piece that finally clicked for me, which I'd been missing when I was taught: HEAD in your repository is just a **pointer**. Doing something like `reset` or changes branches moves the pointer, but the commits themselves don't go anywhere. `reflog` is the trail the pointer left behind, kept for 90 days by default.

The one catch is that it only saves *committed* work, and uncommitted changes blown away by `reset --hard` are gone. Commit early and often!

**When you'd reach for it:** any time you think you've lost work or are in an otherwise irreparable state.

Neat trick: I also learned recently that you can even do things like `main@{one.week.ago}` to go back to where your main branch was a week ago. You don't even need to find exact commits but can say things like "I knew this was working two days ago, let's go back there."

### `git rebase --update-refs`

You have a stacked PR (branch A off main, branch B off A). Someone lands a change on main that conflicts with A. The naive flow is to rebase A onto main, resolve the conflict, then rebase B onto the new A, where the same conflict is waiting for you to resolve a second time.

`git rebase --update-refs main`, run from the top of the stack, rebases the whole stack in one pass and moves the branch pointers along the way. You only need to resolve the conflict once, and then all the downstream branches get updated with the changes from main as well.

**When you'd reach for it:** stacked PRs that should all get updates from a branch at the same time.

**When it shipped:** 2022. I'd originally planned to talk about `git rerere` (it remembers a conflict resolution and replays it next time the same one shows up), and found `--update-refs` while writing the talk.

## The one I think actually matters

If you take one of these home, take reflog.

Worktrees and `--update-refs` are about ergonomics and make day-to-day tasks easier. Reflog is different; it changes the belief that screwing up means losing work, and that belief was what kept me out of the parts of git where the real power lives.

## For the record

A few QA questions where my live answer was either hand-wavy or wrong, because again, I'm not a git expert at all. Here are my corrected answers (and thanks for the opportunity to learn this with you all!)

### `git worktree` vs. multiple clones

Someone asked what the actual difference was between doing `git worktree` vs. just having multiple cloned versions of your repo. A clearer answer than whatever I gave:

Multiple clones give you fully independent copies of the repo, each with its own `.git` directory. Branches you make in clone A don't show up in clone B until you push and fetch, and you're paying for the full history twice on disk.

Worktrees share the underlying `.git` (the secondary worktree has a tiny `.git` *file* pointing back to the main one). Branches and commits show up in every worktree immediately, with no fetching and no duplicate storage. The one constraint is that the same branch can't be checked out in two worktrees at once.

If you want two separate copies of the project that can drift apart, that's `git clone`. If you want one project with multiple working directories so you can sit on two branches at the same time, that's `git worktree`.

### Does reflog work with `reset --soft` and `--hard`?

Yes for both. I also want to walk back something I said on stage, because I had it backwards. Every form of `git reset` moves HEAD, which is the whole point of the command. The flags change what happens to the index and working tree on top of that:

- `--soft`: move HEAD. Index and working tree untouched, so whatever was previously committed shows back up as staged.
- `--mixed` (the default): move HEAD, reset the index to match it. Working tree untouched, so changes show up as unstaged.
- `--hard`: move HEAD, reset the index, and overwrite the working tree to match. Uncommitted changes (staged or not) are gone.

Reflog records every move of HEAD, so all three are recoverable in the sense that you can always put HEAD back where it was. The thing reflog *can't* save you from is the working-tree damage `--hard` does. Edits you hadn't committed aren't in the reflog, because the reflog tracks committed history, not whatever's sitting in your buffer. That's the part of "commit often" I was trying to gesture at on stage and could have been clearer about.

### A note on Jujutsu

At the end of the talk I mentioned [Jujutsu (or `jj`)](https://github.com/jj-vcs/jj). I didn't even realize for a while that there were other version control systems being developed; I kind of thought git was the one we'd settled on. But of course there were VCS systems before and people are making new ones too, revisiting these foundational concepts and/or layering on top.

I find `jj` to be really interesting because it's a more recent VCS that's gaining popularity. It can live alongside a git repo, sharing objects and remotes, but rethinks the model. `jj` has no staging area, and you carry conflicts with you across operations instead of resolving them on the spot. The command surface is also much smaller. I've just heard of it and haven't used it for real work, so I don't have an opinion on whether it's worth switching. But it's on the list, and if you've been bouncing off git for years and it still doesn't fit, that might be the direction to look.

## Coda

I keep coming back to one specific story. After learning software engineering at a bootcamp, I taught there for a year. One day a student ran a rebase even though we'd warned her against it. She lost a day of work and asked for help. The response from the instructors, mine included, was "oops, oh well, we told you so," and I still feel bad about it. Reflog would have walked her right back to where she was, but none of us knew about it, so none of us could tell her. What stuck with her instead was *git is dangerous, stay in the shallow end*.

I gave the talk as a non-expert and that was sort of the point. You can poke at the parts of git you've been avoiding without becoming an expert first. The cost of looking was small for me, a search engine and an afternoon per command, weighed against nine more years of deleting repos and living with what I thought was lost.
