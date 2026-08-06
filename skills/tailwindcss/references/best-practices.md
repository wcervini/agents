# Buenas prácticas e integración (Tailwind v4)

Guía de buenas prácticas de Tailwind v4 y su integración con el stack Astro del usuario. Referencia: https://tailwindcss.com/docs/optimizing-for-production

## Buenas prácticas

### 1. Reutilizar design tokens con @theme

Define colores, fuentes y breakpoints una vez en `@theme` y reutilízalos en todo el proyecto. Evita arbitrary values repetidos.

```css
@theme {
  --color-mint-500: oklch(0.72 0.11 178);
  --font-display: "Satoshi", sans-serif;
}
```

```html
<!-- ✅ Preferir tokens -->
<button class="bg-mint-500 text-white">Acción</button>

<!-- ❌ Evitar arbitrary repetidos -->
<button class="bg-[#3bbf8f] text-white">Acción</button>
```

### 2. No abusar de arbitrary values

Los arbitrary values (`bg-[#...]`, `w-[200px]`) son útiles para casos puntuales, pero si se repiten, conviértelos en tokens de `@theme`. Mejora mantenibilidad y consistencia.

### 3. Tree-shaking del CLI

El CLI de Tailwind **solo emite las utilidades usadas** en tus archivos (escanea clases en templates). No genera CSS de clases no utilizadas.

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./src/output.css
```

> Asegúrate de que el `content`/escaneo cubra todos tus archivos de plantilla (`.astro`, `.tsx`, `.html`, `.vue`, `.js`).

### 4. Dark mode

Usa `dark:` con `@custom-variant` para dark mode class-based:

```css
@custom-variant dark (&:where(.dark, .dark *));
```

```html
<html class="dark">
  <div class="bg-white dark:bg-slate-800">...</div>
</html>
```

### 5. Rendimiento

- Preferir tokens de `@theme` sobre arbitrary values (mejor tree-shaking)
- Usar `@layer` para organizar CSS propio y evitar conflictos de especificidad
- Evitar `@apply` con variantes (no funciona en v4)
- Usar container queries para componentes reutilizables en vez de breakpoints globales

---

## Integración con el stack Astro + Tailwind v4

El proyecto de referencia (**astro-ssr**) usa **Astro + Tailwind v4** con `@tailwindcss/vite` + `@import "tailwindcss"`.

### Configuración

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

@theme {
  --color-mint-500: oklch(0.72 0.11 178);
  --font-display: "Satoshi", sans-serif;
}
```

### Ejemplo de componente Astro con clases Tailwind

```astro
---
// src/components/Card.astro
---
<div class="rounded-2xl bg-white p-6 shadow-sm dark:bg-slate-800">
  <h2 class="font-display text-xl font-bold text-slate-900 dark:text-white">
    Hola, Tailwind v4
  </h2>
  <p class="mt-2 text-sm text-slate-600 dark:text-slate-300">
    Estilado CSS-first con @theme y utilidades.
  </p>
  <button class="mt-4 rounded-lg bg-mint-500 px-4 py-2 text-white hover:bg-mint-600">
    Acción
  </button>
</div>
```

---

## Herramientas MCP disponibles

- **MCP 'Tailwindcss'**: convertir CSS→utilidades, generar paletas de color, generar plantillas de componentes, consultar utilidades y config.
- **MCP 'Context7'**: consultar la documentación oficial de Tailwind (`/tailwindlabs/tailwindcss.com`) para verificar APIs y versiones.

---

## Notas

- Verifica la versión de Tailwind del proyecto vía Context7 antes de dar código.
- No inventes directivas o clases — verifica en tailwindcss.com o vía Context7.