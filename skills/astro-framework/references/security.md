# Security

Astro provides built-in security options under the `security` key in `astro.config.mjs`. These features only apply to pages rendered on demand (SSR) or pages that opt out of prerendering.

**Added in:** Astro 4.9+ (`checkOrigin`), Astro 5.14+ (`allowedDomains`), Astro 5.18+ (`actionBodySizeLimit`), Astro 6.0+ (`csp`, `serverIslandBodySizeLimit`).

## security.checkOrigin

**Type:** `boolean` | **Default:** `true`

Performs a CSRF check that the `origin` header (sent automatically by modern browsers) matches the request URL. It runs only for on-demand pages and only for `POST`, `PATCH`, `DELETE`, and `PUT` requests with `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` content types. On mismatch, Astro returns a `403` and does not render the page.

```javascript
// astro.config.mjs
export default defineConfig({
  output: 'server',
  security: {
    checkOrigin: false, // Disable only when you have a good reason
  },
});
```

> **NEVER** disable `checkOrigin` on public on-demand forms — it is your CSRF protection.

## security.allowedDomains

**Type:** `Array<RemotePattern>` | **Default:** `[]`

Validates the `X-Forwarded-Host` header against a list of permitted host patterns. If the header does not match, it is ignored and the request's original host is used. This prevents host header injection attacks that could manipulate `Astro.url`.

Patterns support `protocol`, `hostname`, and `port` (all validated if provided), with wildcards:
- `*.example.com` — matches exactly one subdomain level
- `**.example.com` — matches any subdomain depth

```javascript
export default defineConfig({
  security: {
    allowedDomains: [
      { hostname: '**.example.com', protocol: 'https' },
      { hostname: 'staging.myapp.com', protocol: 'https', port: '443' },
    ],
  },
});
```

When not configured, `X-Forwarded-Host` headers are not trusted and are ignored.

## security.actionBodySizeLimit

**Type:** `number` | **Default:** `1048576` (1 MB)

Sets the maximum size in bytes allowed for action request bodies. Increase it for actions that accept larger payloads (e.g. file uploads).

```javascript
export default defineConfig({
  security: {
    actionBodySizeLimit: 10 * 1024 * 1024, // 10 MB
  },
});
```

## security.serverIslandBodySizeLimit

**Type:** `number` | **Default:** `1048576` (1 MB)

Sets the maximum size in bytes for server island request bodies (the encrypted props and slot HTML passed to the island). Increase it if your server islands need larger payloads.

```javascript
export default defineConfig({
  security: {
    serverIslandBodySizeLimit: 10 * 1024 * 1024, // 10 MB
  },
});
```

## security.csp

**Type:** `boolean | object` | **Default:** `false`

Enables Content Security Policy support to mitigate XSS. When enabled, Astro adds a `<meta http-equiv="content-security-policy">` element in `<head>` with `script-src` and `style-src` directives based on the page's scripts and styles.

```html
<meta
  http-equiv="content-security-policy"
  content="script-src 'self' 'sha256-somehash'; style-src 'self' 'sha256-somehash';"
/>
```

### Limitations (important)

- **External scripts/styles** are not supported out of the box — provide your own hashes.
- **`<ClientRouter />` (view transitions) is NOT supported** with CSP. Consider the browser-native View Transition API instead.
- **Shiki is NOT supported** — its inline styles conflict with CSP. Use `<Prism />` when you need both CSP and syntax highlighting.
- **`unsafe-inline` is incompatible** — Astro emits hashes for bundled scripts, and modern browsers reject `unsafe-inline` when a hash/nonce is present.
- **Not supported in `dev` mode** — test with `build` and `preview`.

### security.csp.algorithm

**Type:** `"SHA-256" | "SHA-384" | "SHA-512"` | **Default:** `'SHA-256'`

The hash function used for the hashes Astro generates.

```javascript
export default defineConfig({
  security: {
    csp: {
      algorithm: 'SHA-512',
    },
  },
});
```

### security.csp.directives

**Type:** `Array<string>` | **Default:** `[]`

Additional CSP directives (beyond `script-src` and `style-src`) added to all pages.

```javascript
export default defineConfig({
  security: {
    csp: {
      directives: [
        "default-src 'self'",
        "img-src 'self' https://images.cdn.example.com",
      ],
    },
  },
});
```

### security.csp.scriptDirective / styleDirective

Each directive accepts `resources` (override default sources) and `hashes` (additional hashes). Since Astro 7.1, each entry can be a string or an object with a `kind` field:

- `"element"` → stored in `script-src-elem` / `style-src-elem`
- `"attribute"` → stored in `script-src-attr` / `style-src-attr`
- `"default"` → stored in `script-src` / `style-src`

`"attribute"` sources must be one of `'none'`, `'unsafe-hashes'`, `'unsafe-inline'`, or `'report-sample'`. A common use is allowing inline `style` attributes (e.g. from `define:vars` or Shiki) with `{ resource: "'unsafe-inline'", kind: "attribute" }`.

```javascript
export default defineConfig({
  security: {
    csp: {
      scriptDirective: {
        resources: ["'self'", 'https://cdn.example.com'],
        hashes: ['sha384-scriptHash', { hash: 'sha256-scriptHash', kind: 'element' }],
      },
      styleDirective: {
        resources: ["'self'", 'https://styles.cdn.example.com'],
        hashes: ['sha256-styleHash'],
      },
    },
  },
});
```

### security.csp.scriptDirective.strictDynamic

**Type:** `boolean` | **Default:** `false`

Enables the `strict-dynamic` keyword to support dynamic injection of scripts. Applies to `script-src`; when you also scope resources/hashes to `script-src-elem` (via `kind: "element"`), `strict-dynamic` is inherited there.

```javascript
export default defineConfig({
  security: {
    csp: {
      scriptDirective: {
        strictDynamic: true,
      },
    },
  },
});
```

## Best Practices

1. **Keep `checkOrigin` enabled** — CSRF protection for on-demand forms
2. **Configure `allowedDomains`** behind trusted reverse proxies to block host injection
3. **Enable `csp`** and provide hashes for any third-party scripts/styles
4. **Avoid `<ClientRouter />` and Shiki** when CSP is enabled
5. **Raise body size limits only when needed** — keep defaults (1 MB) otherwise
6. **Test CSP with `build` + `preview`** — it is not active in `dev` mode