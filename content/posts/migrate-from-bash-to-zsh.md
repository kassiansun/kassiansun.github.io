---
title: 'Migrate From Bash to Zsh'
date: 2021-11-28T10:14:21+08:00
---

# The story of bash and my taste on command line

I've been using `bash` as the development environment for a long time. The reason for choosing `bash` is its ubiquity.
Most desktop & server Linux distributions and Docker images will ship with bash, which makes it a great fit for a
configure-once-use-everywhere development environment. So you write it once and copy it to every host you'll write
code on. But since ~2014 my bash configuration has degraded several times, for two major reasons:

1. To support different host environments, from Arch Linux to WSL to Cygwin to Mac OS
2. To support different terminal environments, from XShell/iTerm to putty to the serial console
   The need for "use-everywhere" decreased my coding productivity for the most time and made the configuration
   harder and harder to maintain.

Since 2019 I've been a heavy user of Macbook, mainly because
every company ships Macbook for programmers by default. My personal machine was always XPS13, hosting a
Windows + Hyper-V + ArchLinux environment, and using a standard bash environment on ArchLinux, ssh-ed with XShell.
The pros of this configuration:

1. Arch Linux is the most easy-to-use distribution, with rolling updates and a simple packaging strategy.
2. XShell provides a decent good terminal experience.
3. Windows is the most widespread desktop operating system.

The cons are quite obvious: it's hard to configure and hard to port, you need to bring up a brand-new
Arch Linux virtual machine on every new host (the development environment is too big to copy and paste).

On the other hand, Mac OS ships with a shell environment by default, and everything in it is Unix-like.
Along with homebrew, the command line environment feels good enough, although not comparable with the pureness
of Arch Linux. The hardware of Mac OS was always a pain point, but not anymore with the M1 chip. Earlier this year I
purchased an M1 Macbook, after daily usages of more than half a year, I decided to boost my productivity on Mac OS
to maximum, which means the productivity on command line.

# The good parts of `oh-my-zsh` and why it was successful

I've heard of `oh-my-zsh` for several years, `zsh` is not a ubiquity shell, but since I've decided to give up
on the "use-everywhere" feature, it's not a problem anymore. After one-week experience with `oh-my-zsh`, there're
several points I've noticed:

1. The basic `shell` support is similar, which makes porting the old `.bash_profile` very easy.
2. The default configuration of `oh-my-zsh` is good enough, so I don't have to worry about the initial learning curve.
3. The ecosystem (mainly plugins) was a key to success.
4. The `vi-mode` plugin works great, as a heavy user of `vim`, this boosts my keystrokes a lot.
5. Support `django` by default, and I'm learning it to build a backend API server for my personal project.
6. History is synchronized across different terminals, especially useful as I'm always using a lot of tabs for different purposes.

All of these advantages make `oh-my-zsh` a safe suggestion for newbie programmers. It's neither exotic nor bizarre,
it's a decent good starting point and everything you learned with it will be useful later.

# Some pain points as a former bash user

1. `git` plugin shiped by default pollutes the command line with a super long list of `git` shortcuts, as a heavy user of `.gitconfig`,
   it's useless and confusing.
2. `git` has a default integration with `oh-my-zsh`, you need to turn it off. First noticed this when I `cd` into the
   WebKit repository and `zsh` were stuck on it.
3. The completion doesn't always work, many packages don't ship with zsh completion by default.
4. The `tab` behavior feels very different from `bash`, I'm still getting used to it.
5. History across terminals is good, but also a bit confusing when I stoke `up` and see something unexpected.
