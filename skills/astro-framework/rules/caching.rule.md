---
description: Rules for Astro route caching (cache API, route rules, providers)
globs:
  - "astro.config.mjs"
  - "astro.config.ts"
  - "src/pages/**/*.astro"
  - "src/pages/api/**/*.ts"
  - "src/middleware.ts"
---

# Astro Caching Rules

## MUST DO

- Enable caching with `cache: { provider: memoryCache() }` (or a CDN provider per adapter)
- Set default per-route caching with `routeRules` and override per-request with `cache.set()`
- Guard every `Astro.cache.set()` / `context.cache.invalidate()` call behind `cache.enabled`
- Tag responses (`tags`) for targeted invalidation instead of clearing the whole cache
- Use `swr` (stale-while-revalidate) to serve stale content while revalidating in the background
- Use CDN providers on Netlify/Vercel/Cloudflare (`cacheNetlify()`, `cacheVercel()`, `cacheCloudflare()`) to serve from the edge

## MUST NOT DO

- Call `Astro.cache.invalidate()` without a configured provider — it throws
- Call `cache.set()` / `cache.tags` / `cache.options` without checking `cache.enabled` — they log warnings
- Cache personalized responses — opt out with `cache.set(false)`
- Use glob/wildcard patterns in `cache.invalidate({ path })` — path invalidation is exact-match only
- Rely on caching in `dev` mode — `cache.enabled` is always `false` in development

## Example: page caching

```astro
---
// src/pages/products/[id].astro
export const prerender = false; // Not needed in 'server' mode

if (Astro.cache.enabled) {
  Astro.cache.set({ maxAge: 120, swr: 60, tags: ['products'] });
}
---
```

## Example: route rules + provider

```javascript
// astro.config.mjs
import { defineConfig, memoryCache } from 'astro/config';

export default defineConfig({
  cache: {
    provider: memoryCache(),
  },
  routeRules: {
    '/blog/[...path]': { maxAge: 300, swr: 60 },
  },
});
```

See [references/caching.md](../references/caching.md) for the full API.