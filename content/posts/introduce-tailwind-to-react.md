---
title: 'Introducing Tailwind CSS to My React Project'
date: 2021-12-25T11:34:15+08:00
---

### Why Tailwind CSS is good?

Recently, I picked up the frontend project at the current company. Compared with traditional MVC-style native
development, the web development feels more natural to me: declarative, MVVM, state flow, etc.
But if we compare the react/vue with SwiftUI/Jetpack Compose, there's a key pain point: styling.
With SwiftUI/Jetpack Compose, the styling is part of view declaration, which makes it easy to read and maintain.

Traditionally, CSS & HTML are separated from each other, and there're several practices to organize them.
All of these practices have some defects, more or less. In my opinion, all of these issues come from the separation
of CSS and HTML. The author of Tailwind CSS wrote a [great blog](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/) on this.

### Steps to introduce Tailwind CSS

1. Upgrade `react-scripts` to `v5+`, the latest version (v3) of Tailwind CSS depends on a different version of `autoprefix` with `react-scripts@4`.
2. Remove `normalize.css`, Tailwind CSS has its own `Preflight` system: https://tailwindcss.com/docs/preflight.
3. Follow the official guide: https://tailwindcss.com/docs/guides/create-react-app.

Tailwind CSS has a unique naming style, which makes it easy to migrate an old project. Importing
it won't break the existed styles, so I can migrate the old pages one by one.
