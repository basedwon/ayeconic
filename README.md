# ayeconic

An AI-powered, offline semantic Material Design Icon search tool using client-side vector embeddings to natively improve the workflow for frontend developers hunting for the perfect UI icon.

Try it here: [ayeconic live demo](https://basedwon.github.io/ayeconic/)

## Overview

Unlike standard icon search engines that mostly rely on exact names or pre-defined tags, **ayeconic** maps thousands of icons into a semantic vector space in the browser. That lets you search more naturally with concepts like `transportation`, `economy`, `speed`, or `wireless` and get results that are often much closer to what you actually meant.

The project is intentionally simple: a static Vue 2 + Vuetify 2 app with no backend, no database, and no server-side search system.

## Features

* **Semantic search** over Material Design Icons using client-side embeddings
* **Offline-friendly processing** after the initial assets load and cache
* **Blended ranking** using semantic similarity plus keyword scoring
* **Adjustable controls** for tuning search behavior
* **Lazy-loaded results** for large icon sets
* **One-click copy** for final `mdi-*` icon names

## Stack

* Vue 2
* Vuetify 2
* MDI metadata from `@mdi/svg`
* `emlet` for client-side embeddings

## How it works

At a high level, the flow is:

1. load Material Design Icon metadata
2. embed the icon ids locally in the browser
3. embed the user query locally
4. score icons using semantic similarity plus keyword weighting
5. filter and rank the results
6. render results incrementally as you scroll

This gives the app a useful middle ground between plain fuzzy string matching and a heavier remote vector-search setup.

## Why it exists

Normal icon search is usually too literal. If you know the exact icon name, it works. If you only know the idea, it gets annoying fast.

ayeconic exists to make that search process feel more natural for frontend work. The goal is simple: find the right icon faster and move on.

## Local development

Because the app relies heavily on CDNs and browser-native Javascript, there is no bulky compilation required:

1. Clone repo
2. Run `npm install`
3. Run `npm run dev` to start a hot-reloading desktop server.

## License

MIT © basedwon
