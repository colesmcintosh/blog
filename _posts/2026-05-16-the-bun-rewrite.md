---
layout: post
title: "the bun rewrite"
date: 2026-05-16 09:00:00 -0500
categories: update
---

The [Bun](https://bun.sh) team is porting their entire runtime from Zig to Rust. Hundreds of thousands of lines. It's happening fast. And it is absolutely wild.

Five years ago this would have been a multi-year project with a dedicated team and an even chance of never shipping. Rewrites are the graveyard of software — Netscape, the Python 3 migration, every "let's redo it in Rust" thread on hacker news. The default outcome of a rewrite is that it doesn't happen.

Now you can just do it. AI made mass mechanical translation tractable. You point an agent at a file, it produces a faithful translation, you review, you move on. Multiply that by every file in the repo and the thing that used to take person-years takes weeks. The Bun PRs look like a team that woke up one morning, decided they wanted a different language, and went and got one.

The set of changes that used to be "not worth the effort" just collapsed. Switching languages used to be a generational decision — you picked one in 2015 and you lived with it forever. Now it's a project. Same for migrating frameworks, replacing your data layer, splitting a monolith, paying down architectural debt that's been load-bearing since 2017. Calcified decisions all over the industry are about to get unstuck.

And the wild part is this is the worst it'll ever be. The agents doing translation today aren't going to be the ones doing it next year. The bar for what's worth rewriting keeps falling. Most codebases that everyone agreed were "stuck" are about to be unstuck.

I don't know what we're going to do with all of it. But it's incredible to watch happen in real time.
