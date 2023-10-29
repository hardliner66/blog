+++
title = "What About Vim?"
author = ["Steve Biedermann"]
description = "What About Vim?"
date = 2023-02-12T21:13:00+01:00
tags = ["work", "editor", "vim", "neovim", "short"]
categories = ["tooling"]
draft = false
[taxonomies]
  tags = ["work", "editor", "vim", "neovim", "short"]
  categories = ["tooling"]
+++

I talked to a friend of mine about my switch to Emacs and his first question was: But what about vim?


## So, What About Vim? {#so-what-about-vim}

I started using [Neovim](https://neovim.io/) 3 years ago and have been tweaking my config over time to fit my flow, so, why the sudden change?
Well, it's pretty simple to be honest, I like to have everything in one place. Notes, timetracking, editing. The less I have to leave my normal environment,
the less I lose my focus.

My ADHD brain gets distracted easily, which makes it hard to remember all the boring things like logging my time at work or keeping track of certain information.
In the past I used many tools to help me with my daily tasks, but having them separate was enough for me to forget about them. Having everything in one single
tool makes it way simpler to develop a proper workflow, which incorporates all my needs at the same time in a consistent way.

For me, this is basically [org-mode](https://orgmode.org/). It handles my note taking with [org-roam](https://www.orgroam.com/), which also gives me the tools to visualize how those notes link together.
My timetracking issue gets solved by [org-jira](https://github.com/ahungry/org-jira), which allows me to interact with Jira issues, without leaving my editor. And all the knowledge about org-mode I
use at work, I can re-use at home when writing this blog.

Everything else seems to work really well for me too. I can still use my movement, thanks to evil-mode.
Emacs also handles syntax highlighting and stuff like \`jump to definition\` or \`find references\`.

So for now, it's gonna be my editor of choice for most of my work until I hit a roadblock. Then I'll either move back to vim or try one of the many different
editors out there. Maybe I'll someday try to write my own. But thats something for the future.


## Is Vim Dead? {#is-vim-dead}

Not yet. Muscle memory still makes me open Neovim most of the time. Especially for smaller edits and because I have an alias \`v\` which is really fast to type.
I probably will continue using neovim for smaller things, but I probably rework my config to be way smaller and simpler, as most things aren't needed anymore.