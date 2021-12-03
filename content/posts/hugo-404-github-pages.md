---
title: "Hugo on Github Pages Returns 404"
date: 2021-11-22T11:59:11+08:00
tags:
- personal
- blog
- hugo
---

Although followed the official documentation and got it built correctly, the GitHub page still
returns 404. After some investigation with `Branches` and `Pages` settings of my GitHub repository,
I realize that `built`
means a different thing for GitHub. Since the pages are built by Hugo, GitHub doesn't build the
page anymore (unlike the Jekyll I used before). So the source branch shouldn't be `master`, but `gh-pages`, after
switching `Source` to the correct branch, it works now.

So even such a trivial task (get started with Hugo and host it on Github) includes such many problems!
A long time ago I was always struggling with other people confused by programming tasks, because I thought
the solutions are obvious and easy, you just need to do a simple google to solve it. But the "obvious and easy"
part relies heavily on the fundamental knowledge of coding and debugging, you need it to learn new things quickly.
That's why we should always assume other people know nothing before we write/speak anything.
