---
title: "Hugo Extended Mode on Github Pages"
date: 2021-11-22T11:35:34+08:00
---

When I was trying to push this repository to GitHub, by following the [Quick Start](https://gohugo.io/getting-started/quick-start/) and
[Host on GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/), GitHub actions failed to build with this error message:
```text
Error: Error building site: TOCSS: failed to transform "ananke/css/main.css" (text/css). Check your Hugo installation; you need the extended version to build SCSS/SASS.: this feature is not available in your current Hugo version, see https://goo.gl/YMrWcn for more information
```

So the answer is simple, comment the `#extended: true` in `.github/workflows/gh-pages.yml`, the extended version of Hugo will
support building scss/sass. The official documentation didn't mention any on this configuration, so I guess this is a too simple
question to mention on the document, just a commented line of correct configuration should be good enough to let people know this.

But unfortunately, I didn't notice it, googled and got an ambiguous answer on `using the extended version`, :(. The good thing is
I got something to write on the first day of building this blog :).
