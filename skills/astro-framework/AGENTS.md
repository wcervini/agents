# Astro Framework Cheatsheet

**Version:** 3.0.0 | **Astro 7.x** | **Updated:** 2026-08-06 | **Author**: [webreactiva.com](https://webreactiva.com/ia)

---

## Quick Reference Card

| Topic | Rule of Thumb |
|-------|---------------|
| **Components** | Define `interface Props`; frontmatter = server only; use `class:list` for conditional classes |
| **Hydration** | Default: no directive (zero JS). Question every `client:` |
| **Server Islands** | `server:defer` for personalized/dynamic server content on cached pages |
| **Content Collections** | Content Layer API with loaders in `src/content.config.ts` (NOT `src/content/config.ts`) |
| **Rendering content** | `import { render } from 'astro:content'` then `const { Content } = await render(entry)` |
| **Routing** | File-based; `getStaticPaths()` required for dynamic routes in static mode |
| **Output modes** | `static` (default), `server` (all SSR), `hybrid` (static default, opt-in SSR) |
| **Sessions** | `Astro.session?.get/set`; always use optional chaining; SSR only |
| **Env vars** | `astro:env` schema with `envField` for type safety; import from `astro:env/client` or `astro:env/server` |
| **Images** | Import local images; use `<Image>` / `<Picture>` from `astro:assets`; always set alt text |
| **i18n** | Use `getRelativeLocaleUrl()` for links, never hardcode locale prefixes |
| **TypeScript** | Extend `astro/tsconfigs/strict`; define path aliases; type `App.Locals` in `src/env.d.ts` |
| **API routes** | Export named handlers (`GET`, `POST`, etc.) typed as `APIRoute` |
| **Security** | `security.csp` for CSP; `checkOrigin` guards CSRF; `allowedDomains` blocks host injection; body limits protect actions/server islands |
| **Caching** | `cache: { provider: memoryCache() }` + `routeRules` (Astro 7); guard `cache.set()`/`invalidate()` behind `cache.enabled` |
| **Advanced routing** | `src/fetch.ts` overrides the request pipeline; `astro/fetch` handlers + `FetchState` (Astro 7) |

---

## Decision Trees

### Output Mode
```
All static? ──> output: 'static' (default, no adapter)
All dynamic? ──> output: 'server' + adapter
Mix? ──> output: 'hybrid' + adapter
  Opt into SSR (hybrid): export const prerender = false
  Opt out of SSR (server): export const prerender = true
```

### Hydration Directive
```
Needs interactivity?
├─ No ──> No directive (zero JS)
└─ Yes ──> Above fold + critical? ──> client:load
            Above fold + non-critical? ──> client:idle
            Below fold? ──> client:visible
            Device-specific? ──> client:media="(query)"
            Browser APIs only, skip SSR? ──> client:only="react"
```

### Server Island vs Client Island
```
Needs server data (cookies, DB, personalization)? ──> server:defer + slot="fallback"
Needs browser interactivity? ──> client:* directive
Neither? ──> Static .astro component
```

---

## Critical "NEVER Do" List

- **NEVER** access `window`/`document` in `.astro` frontmatter (server-only)
- **NEVER** use `client:load` on everything — defeats islands architecture
- **NEVER** use `client:only` without the framework string argument
- **NEVER** use plain `z.date()` for frontmatter dates — use `z.coerce.date()`
- **NEVER** use string paths for local images (`src="/images/hero.jpg"`) — import them
- **NEVER** access `Astro.request`/`Astro.cookies`/`Astro.session` in prerendered pages
- **NEVER** rely on `Astro.url` inside a server island (returns internal route; use `Referer` header)
- **NEVER** pass large objects/functions as server island props (IDs only; >2048 bytes forces POST)
- **NEVER** use sessions in edge middleware or static pages
- **NEVER** forget `getStaticPaths()` for `[param]` routes in static output
- **NEVER** skip `slot="fallback"` on `server:defer` components
- **NEVER** use `any` — use `unknown` and narrow
- **NEVER** import Zod from `zod` — use `astro/zod` (Astro 5+)
- **NEVER** call `Astro.cache.invalidate()`/`cache.set()` without checking `cache.enabled` (throws/`warns` without a provider)
- **NEVER** use `<ClientRouter />` (view transitions) or Shiki with `security.csp` enabled — incompatible
- **NEVER** disable `security.checkOrigin` on public on-demand forms — it provides CSRF protection
- **NEVER** keep `src/fetch.ts` for non-routing code in Astro 7 — it's a reserved entrypoint (`fetchFile: null` to disable)

---

## Astro 5+ Migration Notes (Post-Training Knowledge)

### Content Collections (Content Layer API)

- Config file: `src/content.config.ts` (was `src/content/config.ts`)
- Uses `loader` property instead of `type: 'content'` / `type: 'data'`
- Loaders: `import { glob, file } from 'astro/loaders'`
- Zod: `import { z } from 'astro/zod'` (NOT from `zod` package)
- Rendering: `import { render } from 'astro:content'` then `await render(entry)` (was `entry.render()`)
- `reference()` for cross-collection relationships; `image()` schema helper for optimized images

### server:defer (Stable in Astro 5)

- Requires adapter; props must be serializable and small
- Use `Referer` header inside island to get parent page URL

### Sessions (Astro 5.7+)

- `Astro.session?.get(key)` / `.set(key, value)` — always optional-chain
- Configure `session.driver` in config via `sessionDrivers` from `astro/config`
- `session.regenerate()` after login, `session.destroy()` on logout
- Type via `App.SessionData` in `src/env.d.ts`

### astro:env (Stable in Astro 5)

- Schema in `astro.config.mjs` with `envField` from `astro/config`
- Import from `astro:env/client` or `astro:env/server`
- Three kinds: public client (bundled), public server (server bundle), secret server (not bundled)
- No secret client vars — not supported by design

### Actions

- `defineAction` from `astro:actions` with `astro/zod` schemas

## Astro 6+ / Astro 7 Notes (Post-Training Knowledge)

### Live Loaders (Astro 6+)

- Live collections live in `src/live.config.ts` (NOT `src/content.config.ts`) using `defineLiveCollection()` from `astro:content`
- Live loaders implement `loadCollection()` / `loadEntry()` (not `load`); return data directly, no data store
- Query with `getLiveCollection()` / `getLiveEntry()`; requires an adapter (on-demand rendering)
- Type with `LiveLoader<TData, TEntryFilter, TCollectionFilter, TError>`; errors: `LiveCollectionError`, `LiveEntryNotFoundError`, `LiveCollectionValidationError`
- Live loaders can return a `cacheHint` (tags + lastModified) — pass it to `Astro.cache.set()` (Astro 7)

### Security (Astro 6+)

- `security.csp` (boolean | object): `algorithm`, `directives`, `scriptDirective`/`styleDirective` with `resources`/`hashes`/`kind`, `strictDynamic`
- CSP is incompatible with `<ClientRouter />` and Shiki; not supported in `dev` mode
- `security.checkOrigin` (default `true`) — CSRF protection for on-demand forms
- `security.allowedDomains` — validates `X-Forwarded-Host` to block host header injection
- `security.actionBodySizeLimit` (default 1 MB) and `security.serverIslandBodySizeLimit` (default 1 MB)

### Route Caching (Astro 7)

- Enable with `cache: { provider: memoryCache() }`; set defaults via `routeRules: { '/path': { maxAge, swr } }`
- `Astro.cache` / `context.cache`: `set()`, `invalidate()`, `enabled`, `options`, `tags`
- CDN providers per adapter: `cacheNetlify()` (`@astrojs/netlify/cache`), `cacheVercel()` (`@astrojs/vercel/cache`), `cacheCloudflare()` (`@astrojs/cloudflare/cache`)

### Advanced Routing (Astro 7)

- `src/fetch.ts` (default-export `{ fetch(request) }`) replaces the default request pipeline
- Handlers from `astro/fetch`: `astro`, `actions`, `cache`, `i18n`, `middleware`, `pages`, `redirects`, `sessions`, `trailingSlash`; state via `new FetchState(request)`
- Hono middleware available from `astro/hono`; rename/disable entrypoint with `fetchFile`

## Key Patterns (Brief)

- **Component structure:** imports, `interface Props`, destructure with defaults, logic, template, scoped `<style>`
- **Slots:** `Astro.slots.has('name')` to conditionally render named slot wrappers
- **Middleware:** `defineMiddleware` from `astro:middleware`; chain with `sequence(a, b)`; file: `src/middleware.ts`
- **Pagination:** `getStaticPaths({ paginate })` returns `page.data`, `page.url.prev/next`
- **Redirects:** `redirects: { '/old': '/new' }` in `astro.config.mjs`
- **Cookies (SSR):** `Astro.cookies.get('name')?.value` / `.set('name', value, options)`

## References

Deep-dive docs with full code examples live in `references/`:

`actions.md` `advanced-routing.md` `caching.md` `client-directives.md` `components.md` `configuration.md` `content-collections.md` `environment-variables.md` `i18n-routing.md` `images.md` `middleware.md` `routing.md` `security.md` `server-islands.md` `sessions.md` `ssr-adapters.md` `styling.md` `view-transitions.md`
