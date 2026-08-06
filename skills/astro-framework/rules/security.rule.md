---
description: Rules for Astro security configuration (CSP, origin checks, body limits)
globs:
  - "astro.config.mjs"
  - "astro.config.ts"
  - "src/middleware.ts"
---

# Astro Security Rules

## MUST DO

- Keep `security.checkOrigin` enabled (default `true`) on on-demand forms — it provides CSRF protection
- Configure `security.allowedDomains` when behind a trusted reverse proxy to block `X-Forwarded-Host` host header injection
- Enable `security.csp` and provide hashes for any third-party scripts/styles
- Use `security.actionBodySizeLimit` and `security.serverIslandBodySizeLimit` only when you genuinely need larger payloads
- Authorize actions from the handler (`ActionError` with `UNAUTHORIZED`) or gate them in middleware with `getActionContext()`
- Test CSP with `build` + `preview` (it is not active in `dev` mode)

## MUST NOT DO

- Disable `security.checkOrigin` on public on-demand forms
- Use `<ClientRouter />` (view transitions) or Shiki when `security.csp` is enabled — incompatible
- Rely on `unsafe-inline` with CSP — Astro emits hashes and browsers reject `unsafe-inline` alongside them
- Trust `X-Forwarded-Host` without `allowedDomains` — it can be spoofed to manipulate `Astro.url`
- Raise body size limits beyond what the app actually needs (defaults are 1 MB)

## Example: full security config

```javascript
// astro.config.mjs
export default defineConfig({
  output: 'server',
  security: {
    checkOrigin: true,
    allowedDomains: [{ hostname: '**.example.com', protocol: 'https' }],
    actionBodySizeLimit: 10 * 1024 * 1024,
    serverIslandBodySizeLimit: 10 * 1024 * 1024,
    csp: {
      algorithm: 'SHA-256',
      directives: ["default-src 'self'", "img-src 'self' https://images.cdn.example.com"],
      scriptDirective: {
        resources: ["'self'", 'https://cdn.example.com'],
        hashes: ['sha384-scriptHash'],
      },
      styleDirective: {
        resources: ["'self'"],
      },
    },
  },
});
```

See [references/security.md](../references/security.md) for the full API.