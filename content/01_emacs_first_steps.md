+++
title = "First Steps in Emacs"
author = ["Steve Biedermann"]
description = "How my first days with Emacs went."
date = 2023-02-10T00:00:00+01:00
tags = ["work", "editor", "short"]
categories = ["tooling"]
draft = false
[taxonomies]
  tags = ["work", "editor", "short"]
  categories = ["tooling"]
+++

In my time as dev, I used many different editors and IDEs. Notepad for QBasic, Excel Macro Editor for VBA and many more.
A few years ago, I switched to Neovim and thought that's it, I thought I found my favorite text editor, but oh boy was I wrong.


## Why Emacs? {#why-emacs}

So, why Emacs you might ask and what's so great about it? Well, I want to learn coding in Common Lisp and there is almost no way around Emacs for that.
But it's not only Lisp I was interested in, I also wanted to try out org-mode and let me tell you, org-mode alone is worth the hassle of learning a new editor.
You can use it for anything from note taking, over blogging ([this blog](https://github.com/hardliner66/blog) is written in org-mode) to full blown time tracking.

So if you're like me and always forget to track your time at work, there even is an org-mode plugin called [org-jira](https://github.com/ahungry/org-jira), which can not only fetch your issues from a jira server,
it also can sync your changes back to jira. Now I finally don't have to switch software for bookkeeping.


## But what about the config? {#but-what-about-the-config}

I went with Doom Emacs as my base to speed things up and only added a few small changes to the config so far and I currently have most things I had in my Neovim config and more.


## And the coding? {#and-the-coding}

At work we use Yocto, which is a framework to build custom Linux distributions.
One of the major drawbacks of it is, that it uses its own build system on top of all the build systems that the packages use.
Neovim LSP doesn't like that at all. Even COC struggled most of the time. Emacs gets at least the basics somewhat working.

So, it's still not perfect, but it's better than it was before.


## Should you change too? {#should-you-change-too}

Well, probably not. If you're happy with your editor or have special requirements, Emacs might not be the tool for you.
Its graphic nature means, you can't just run it on a terminal and it's much more heavy weight than a basic Neovim setup. Which might be a problem for some.

If you're still searching for the right editor for you or you're curious about org-mode (you should be) then give it at least a try.
Install Doom Emacs and start using org-mode. And if org-mode is too much at first, try [org-roam](https://www.orgroam.com/) instead.
Org-roam gives you a nice local knowledgebase system, which can also show you back-links and generate nice graphs.


## Conclusion {#conclusion}

It's only been a week since I started using it, so I can't give a good final conclusion yet, but for now, I'm impressed at the things Emacs lets me do with less than a week of experience.