---
description: Reglas de configuración CSS-first y design tokens con Tailwind v4
globs:
  - "**/*.css"
  - "**/*.astro"
  - "**/*.vue"
  - "**/*.jsx"
  - "**/*.tsx"
---

# Tailwind CSS-first Rules

## MUST DO

- Usar `@import "tailwindcss";` como punto de entrada (v4 CSS-first)
- Configurar design tokens con `@theme` en vez de inline arbitrary values repetidos
- Usar `@layer base/components/utilities` para organizar CSS propio
- Usar `@apply` para reutilizar utilidades dentro de componentes CSS
- Preferir tokens de `@theme` sobre arbitrary values (`bg-mint-500` en vez de `bg-[#...]`)
- En Astro, instalar con `@tailwindcss/vite` y añadir el plugin en `astro.config.mjs`
- Verificar directivas dudosas en tailwindcss.com o vía Context7

## MUST NOT DO

- No crear `tailwind.config.js` en v4 — la configuración es CSS-first
- No usar `@tailwind base/components/utilities` (v3) — en v4 es `@import "tailwindcss"`
- No abusar de arbitrary values (`[#...]`) cuando existe un token en `@theme`
- No usar `@apply` con clases que dependen de variantes (no funciona en v4)
- No inventar directivas o clases sin verificar en tailwindcss.com

## Patrones

### Configuración CSS-first

```css
@import "tailwindcss";

@theme {
  --color-mint-500: oklch(0.72 0.11 178);
  --font-display: "Satoshi", sans-serif;
  --breakpoint-3xl: 120rem;
}
```

### @layer + @apply

```css
@layer components {
  .btn-primary {
    @apply rounded-lg bg-mint-500 px-4 py-2 text-white hover:bg-mint-600;
  }
}
```

### Dark mode class-based

```css
@custom-variant dark (&:where(.dark, .dark *));
```