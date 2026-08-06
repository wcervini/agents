# Instalación de Tailwind CSS v4 por framework

Guía de instalación de Tailwind CSS v4 (CSS-first) en distintos frameworks. Referencia: https://tailwindcss.com/docs/installation

## Tabla resumen: framework → paquete → configuración

| Framework | Paquete | Configuración | Import |
|-----------|---------|---------------|--------|
| Vite | `tailwindcss` + `@tailwindcss/vite` | plugin `tailwindcss()` en `vite.config` | `@import "tailwindcss"` |
| Astro | `tailwindcss` + `@tailwindcss/vite` | `@tailwindcss/vite` en `astro.config.mjs` (vite.plugins) | `@import "tailwindcss"` en CSS global |
| Next.js | `tailwindcss` + `@tailwindcss/postcss` + `postcss` | `"@tailwindcss/postcss": {}` en `postcss.config.mjs` | `@import "tailwindcss"` en globals.css |
| Nuxt | `tailwindcss` + `@tailwindcss/vite` | plugin Vite en `nuxt.config` | `@import "tailwindcss"` |
| Node/Express/Hono | `tailwindcss` + `@tailwindcss/cli` | build standalone (o vía Vite) | `@import "tailwindcss"` |
| Sin bundler | `tailwindcss` + `@tailwindcss/cli` | CLI standalone | `@import "tailwindcss"` |

---

## Vite

```bash
npm i tailwindcss @tailwindcss/vite
```

```js
// vite.config.js
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [tailwindcss()],
});
```

```css
/* src/index.css */
@import "tailwindcss";
```

---

## Astro

```bash
npm i tailwindcss @tailwindcss/vite
```

```js
// astro.config.mjs
import { defineConfig } from "astro/config";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
});
```

```css
/* src/styles/global.css */
@import "tailwindcss";
```

> **Stack de referencia (astro-ssr):** usa `@tailwindcss/vite` + `@import "tailwindcss"`. No uses el paquete `@astrojs/tailwind` (obsoleto en v4).

---

## Next.js

```bash
npm i tailwindcss @tailwindcss/postcss postcss
```

```js
// postcss.config.mjs
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
export default config;
```

```css
/* app/globals.css */
@import "tailwindcss";
```

---

## Nuxt

```bash
npm i tailwindcss @tailwindcss/vite
```

```ts
// nuxt.config.ts
import tailwindcss from "@tailwindcss/vite";

export default defineNuxtConfig({
  vite: {
    plugins: [tailwindcss()],
  },
});
```

```css
/* assets/css/main.css */
@import "tailwindcss";
```

---

## Node / Express / Hono (standalone)

Sin bundler, usa el CLI standalone:

```bash
npm i -D tailwindcss @tailwindcss/cli
```

```bash
# Compilar el CSS
npx @tailwindcss/cli -i ./src/input.css -o ./public/output.css --watch
```

```css
/* src/input.css */
@import "tailwindcss";
```

> Alternativa: montar Vite en el proyecto y usar `@tailwindcss/vite` como en la sección Vite.

---

## Notas

- En **v4 NO existe `tailwind.config.js`** — la configuración es CSS-first (`@theme`, `@layer`, `@apply`).
- El punto de entrada siempre es `@import "tailwindcss";` (reemplaza a `@tailwind base/components/utilities` de v3).
- Verifica la versión del proyecto vía Context7 antes de dar código.