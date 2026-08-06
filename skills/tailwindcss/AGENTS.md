# Tailwind CSS v4 Cheatsheet

**Version:** 1.0.0 | **Tailwind CSS v4 (CSS-first)** | **Updated:** 2026-08-06 | **Author**: delineas

---

## Quick Reference Card

| Tema | Regla práctica |
|------|---------------|
| **Concepto v4** | **CSS-first configuration** — NO hay `tailwind.config.js`; config en CSS con `@import "tailwindcss"` + `@theme`/`@layer`/`@apply`/`@variant` |
| **Vite / Astro / Nuxt** | `npm i tailwindcss @tailwindcss/vite` + plugin `tailwindcss()` en `vite.config` / `astro.config.mjs` |
| **Next.js** | `npm i tailwindcss @tailwindcss/postcss postcss` + `"@tailwindcss/postcss": {}` en `postcss.config.mjs` |
| **Node/Express/Hono** | `npm i -D tailwindcss @tailwindcss/cli` → build standalone (o vía Vite) |
| **Import** | `@import "tailwindcss";` en el CSS global (reemplaza a `@tailwind base/components/utilities` de v3) |
| **Design tokens** | `@theme { --color-mint-500: oklch(...); --font-display: ...; --breakpoint-3xl: ...; --ease-snappy: ... }` → genera utilidades + custom properties |
| **Capas** | `@layer base/components/utilities { ... }` para CSS propio organizado |
| **Reutilizar utilidades** | `@apply flex items-center gap-2` dentro de una clase CSS |
| **Variantes** | `@variant` (nueva) y `@custom-variant` (p. ej. dark class-based) |
| **Responsive** | `sm:` `md:` `lg:` `xl:` `2xl:` + breakpoints custom (`--breakpoint-3xl`) |
| **Dark mode** | `dark:` por defecto (media query); class-based: `@custom-variant dark (&:where(.dark, .dark *))` |
| **Container queries** | `@container` en el padre + `@sm:` `@md:` `@8xl:` en los hijos |
| **Estado moderno** | `:user-valid`, `:user-invalid`, `:placeholder-shown`, `:has()` |
| **Plugins oficiales** | `@tailwindcss/typography` (`prose`), `@tailwindcss/forms`, `@tailwindcss/container-queries`, `@tailwindcss/postcss` (Next) |
| **Arbitrary values** | `bg-[#123456]` — usar con moderación; preferir tokens de `@theme` |
| **Rendimiento** | El CLI solo emite lo usado (tree-shaking); no abusar de arbitrary values |
| **Stack Astro** | `@tailwindcss/vite` en `astro.config.mjs` + `@import "tailwindcss"` en CSS global |
| **MCP** | 'Tailwindcss' (convert CSS→utilities, paletas, plantillas) + 'Context7' (doc oficial) |
| **Ref. raíz** | https://tailwindcss.com/docs |

---

## Decision Trees

### v3 vs v4
```
¿El proyecto usa Tailwind v4 (CSS-first)?
├── Sí → NO tailwind.config.js → @theme/@layer/@apply en CSS
└── No (v3) → tailwind.config.js con theme.extend + plugins
```

### Integración por framework
```
¿Qué framework usa el proyecto?
├── Vite / Astro / Nuxt → @tailwindcss/vite (plugin Vite)
├── Next.js             → @tailwindcss/postcss (postcss.config.mjs)
├── Node/Express/Hono   → @tailwindcss/cli (standalone) o vía Vite
└── Sin bundler         → @tailwindcss/cli
```

### @theme vs @layer vs @apply
```
¿Definir un design token reutilizable?
├── Sí → @theme { --color-mint-500: oklch(...) }  → utilidad + custom property
├── ¿CSS base (reset/global)? → @layer base { ... }
├── ¿Reutilizar utilidades en una clase? → @apply flex items-center
└── ¿Variante personalizada? → @variant / @custom-variant
```

---

## Critical "NEVER Do" List

- **NEVER** crear `tailwind.config.js` en v4 — la configuración es CSS-first
- **NEVER** usar `@tailwind base/components/utilities` (v3) — en v4 es `@import "tailwindcss"`
- **NEVER** abusar de arbitrary values (`[#...]`) cuando existe un token en `@theme`
- **NEVER** inventar directivas o clases — verifica en tailwindcss.com o Context7
- **NEVER** usar `@apply` con clases que dependen de variantes (no funciona en v4)
- **NEVER** olvidar añadir `@tailwindcss/vite` en `astro.config.mjs`/`vite.config`
- **NEVER** confundir `@tailwindcss/postcss` (Next) con `@tailwindcss/vite` (Vite/Astro/Nuxt)

---

## Tailwind v4 Notes (Post-Training / Verificado con Context7)

### CSS-first configuration — @theme

En v4 la configuración vive en el CSS. `@theme` define design tokens que Tailwind convierte automáticamente en utilidades y custom properties:

```css
@import "tailwindcss";

@theme {
  --font-display: "Satoshi", "sans-serif";

  --breakpoint-3xl: 1920px;

  --color-avocado-100: oklch(0.99 0 0);
  --color-avocado-500: oklch(0.84 0.18 117.33);
  --color-avocado-600: oklch(0.53 0.12 118.34);

  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
  --ease-snappy: cubic-bezier(0.2, 0, 0, 1);
}
```

- `--color-*` → genera `bg-*`, `text-*`, `border-*`, etc.
- `--font-*` → genera `font-*`
- `--breakpoint-*` → genera variantes responsive (`3xl:`)
- `--ease-*` → genera `ease-*`
- Los tokens también se exponen como custom properties CSS (`var(--color-mint-500)`)

### Dark mode class-based

Por defecto `dark:` usa `prefers-color-scheme`. Para dark mode por clase `.dark`:

```css
@custom-variant dark (&:where(.dark, .dark *));
```

### Container queries

```html
<div class="@container">
  <div class="flex flex-col @8xl:flex-row">
    <!-- ... -->
  </div>
</div>
```

### Plugins oficiales

```js
// tailwind.config.js — SOLO para CLI standalone (v4 no usa config en Vite/Astro)
module.exports = {
  plugins: [require("@tailwindcss/forms"), require("@tailwindcss/typography")],
};
```

> En v4 con Vite/Astro, los plugins se registran vía CSS `@plugin` (ver `references/plugins.md`).

---

## Integraciones (mínimas)

| Integración | Paquete | Configuración |
|-------------|---------|---------------|
| Vite | `@tailwindcss/vite` | `plugins: [tailwindcss()]` en `vite.config` |
| Astro | `@tailwindcss/vite` | `vite: { plugins: [tailwindcss()] }` en `astro.config.mjs` + `@import "tailwindcss"` |
| Next.js | `@tailwindcss/postcss` | `"@tailwindcss/postcss": {}` en `postcss.config.mjs` + `@import "tailwindcss"` en globals.css |
| Nuxt | `@tailwindcss/vite` | plugin Vite en `nuxt.config` |
| Node/Express/Hono | `@tailwindcss/cli` | build standalone del CSS |
| Sin bundler | `@tailwindcss/cli` | `npx @tailwindcss/cli -i input.css -o output.css` |

---

## Key Patterns (Brief)

- **`@import "tailwindcss"`** — punto de entrada v4 (reemplaza a `@tailwind base/components/utilities`)
- **`@theme`** — define design tokens → utilidades + custom properties
- **`@layer base/components/utilities`** — organiza CSS propio
- **`@apply`** — reutiliza utilidades dentro de una clase CSS
- **`@variant`** — define variantes personalizadas; **`@custom-variant`** — dark class-based
- **Responsive** — `sm:` `md:` `lg:` `xl:` `2xl:` + breakpoints custom
- **Dark** — `dark:` (media query por defecto) o `@custom-variant dark`
- **Container** — `@container` en padre + `@sm:`/`@md:`/`@8xl:` en hijos
- **Estado** — `:user-valid`, `:user-invalid`, `:placeholder-shown`, `has-*`
- **Arbitrary** — `bg-[#5]`, `w-[200px]` — usar con moderación, preferir tokens
- **Stack Astro** — `@tailwindcss/vite` + `@import "tailwindcss"` (astro-ssr ya lo usa)

## References

Docs a detalle con ejemplos completos en `references/`:

`installation.md` `css-first-config.md` `utilities.md` `plugins.md` `best-practices.md`