# ayeconic

An AI-powered, offline semantic Material Design Icon search tool using client-side vector embeddings to natively improve the workflow for frontend developers hunting for the perfect UI icon.

## Overview
Unlike standard icon search engines that only filter by an exact name or pre-determined alias tags, **ayeconic** mathematically maps thousands of icons into a 96-dimension semantic space. This allows you to search for concepts naturally (e.g. typing "transportation", "economy", or "speed") and immediately receive the most contextually relevant Material Design icons.

## Features
- **Semantic ML Search**: Automatically pulls `emlet` to run complex cosine similarity vector math without making a single third-party server call.
- **Client-Side Offline Processing**: 100% offline-capable after the initial payload caches. 
- **Infinite Smooth Scroll**: Renders thousands of mapped icons performantly into the browser DOM using Intersection Observers.
- **Cyberpunk Aesthetics**: Hand-curated electric styling wrapped around a responsive Vuetify engine with a Zero-FOUC (Flash of Unstyled Content) loader.
- **One-Click Copy**: Tapping an icon silently copies the fully formatted Vue class implementation (e.g. `mdi-console-network`) to your clipboard.

## Local Development
Because the app relies heavily on CDNs and browser-native Javascript, there is no bulky compilation required:

1. Clone repo
2. Run `npm install`
3. Run `npm run dev` to start a hot-reloading desktop server.

## License
MIT © basedwon
