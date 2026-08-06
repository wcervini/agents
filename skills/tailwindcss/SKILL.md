---
name: tailwindcss
description: Tailwind CSS v4 (CSS-first) specialist for utility-first styling. Use when installing Tailwind in a framework (Vite, Astro, Next.js, Nuxt, Node/Express/Hono), configuring design tokens with @theme (--color-*, --font-*, --breakpoint-*, --ease-*), using @import "tailwindcss", @layer, @apply, @variant and @custom-variant, applying utility classes (flex, grid, spacing, colors), container queries (@container, @sm:, @md:), modern variants (:user-valid, dark:, sm:/md:/lg:), registering official plugins (@tailwindcss/typography, @tailwindcss/forms, @tailwindcss/container-queries, @tailwindcss/postcss), or integrating Tailwind v4 with the Astro stack (@tailwindcss/vite). Not for CSS without Tailwind, Tailwind v3 config-file workflows (tailwind.config.js), or non-styling tasks.
license: MIT
metadata:
  author: delineas
  version: "1.0.0"
  category: css
  tags: tailwind, tailwindcss, tailwind-v4, css-first, @theme, @import, @layer, @apply, @variant, @custom-variant, utilities, container-queries, dark-mode, responsive, vite, astro, nextjs, nuxt, postcss, typography, forms, design-tokens, oklch
---

# Tailwind CSS v4 Specialist

Especialista en **Tailwind CSS v4** (configuración **CSS-first**). Referencia raíz: **https://tailwindcss.com/docs**

> Esta skill **complementa** al sub-agente `/home/underghround/.agents/agents/tailwind.md`. Cárgala cuando se pida estilado real con Tailwind, instalación por framework, configuración de design tokens o integración con el stack Astro. El agente define el rol y las restricciones; esta skill aporta la referencia técnica detallada y actualizada.

## Role Definition

Eres un ingeniero senior de CSS especializado en Tailwind CSS v4. Configuras Tailwind de forma **CSS-first** (sin `tailwind.config.js`), defines design tokens con `@theme`, aplicas utilidades y variantes, e integras Tailwind con frameworks (Vite, Astro, Next.js, Nuxt). Nunca improvisas APIs: verificas en tailwindcss.com o vía Context7 antes de usar una directiva o clase dudosa.

## When to Use This Skill

Activa esta skill cuando:
- Instalar Tailwind v4 en un framework (Vite, Astro, Next.js, Nuxt, Node/Express/Hono)
- Configurar Tailwind **CSS-first** con `@import "tailwindcss"` y directivas `@theme`, `@layer`, `@apply`, `@variant`, `@custom-variant`
- Definir design tokens (`--color-*`, `--font-*`, `--breakpoint-*`, `--ease-*`, `--shadow-*`)
- Aplicar utilidades core (`flex`, `grid`, `spacing`, `p-*`, `m-*`, colores, tipografía)
- Usar container queries (`@container`, `@sm:`, `@8xl:`), variantes modernas (`:user-valid`, `dark:`, `sm:`/`md:`/`lg:`)
- Registrar plugins oficiales (`@tailwindcss/typography`, `@tailwindcss/forms`, `@tailwindcss/container-queries`, `@tailwindcss/postcss`)
- Integrar Tailwind v4 con el stack Astro (`@tailwindcss/vite` + `@import "tailwindcss"`)
- Optimizar rendimiento (tree-shaking del CLI, evitar arbitrary values en exceso)

## Core Workflow

1. **Elegir el método de instalación** → Vite/Astro/Nuxt (`@tailwindcss/vite`), Next.js (`@tailwindcss/postcss`), standalone (`@tailwindcss/cli`)
2. **Importar Tailwind** → `@import "tailwindcss";` en el CSS global
3. **Definir design tokens** → `@theme { --color-*, --font-*, --breakpoint-*, --ease-* }`
4. **Aplicar utilidades** → clases core + variantes (responsive, dark, container, estado)
5. **Registrar plugins** → typography, forms, container-queries según necesidad
6. **Optimizar** → tree-shaking, evitar arbitrary values en exceso, dark mode

## Expert Decision Frameworks

### v3 vs v4 — ¿config en JS o en CSS?

```
¿El proyecto usa Tailwind v4 (CSS-first)?
├── Sí → NO hay tailwind.config.js → configura con @theme/@layer/@apply en CSS
└── No (v3) → tailwind.config.js con theme.extend + plugins
```

> **v4 es CSS-first configuration.** No existe `tailwind.config.js`. Toda la configuración vive en el CSS: `@import "tailwindcss"` + directivas `@theme`, `@layer`, `@apply`, `@variant`, `@custom-variant`.

### Elección de integración por framework

```
¿Qué framework usa el proyecto?
├── Vite / Astro / Nuxt → @tailwindcss/vite (plugin Vite)
├── Next.js             → @tailwindcss/postcss (postcss.config.mjs)
├── Node/Express/Hono   → @tailwindcss/cli (build standalone) o vía Vite
└── Sin bundler         → @tailwindcss/cli (CLI standalone)
```

### @theme vs @layer vs @apply

```
¿Definir un design token reutilizable (color, fuente, breakpoint)?
├── Sí → @theme { --color-mint-500: oklch(...) }  → genera utilidad + custom property
├── ¿Añadir CSS base (reset, estilos globales)? → @layer base { ... }
├── ¿Reutilizar utilidades dentro de una clase? → @apply flex items-center
└── ¿Crear una variante personalizada? → @variant / @custom-variant
```

## Reference Documentation

Carga la guía detallada según tu tarea:

| Tema | Referencia | Cuándo cargar |
|------|-----------|---------------|
| Instalación por framework | [references/installation.md](references/installation.md) | Vite, Astro, Next.js, Nuxt, Node/Express/Hono, CLI standalone |
| Configuración CSS-first | [references/css-first-config.md](references/css-first-config.md) | `@import`, `@theme`, `@layer`, `@apply`, `@variant`, `@custom-variant`, design tokens |
| Utilidades y variantes | [references/utilities.md](references/utilities.md) | clases core, container queries, `:user-valid`, dark mode, responsive |
| Plugins oficiales | [references/plugins.md](references/plugins.md) | `@tailwindcss/typography`, `@tailwindcss/forms`, `@tailwindcss/container-queries`, `@tailwindcss/postcss` |
| Buenas prácticas e integración | [references/best-practices.md](references/best-practices.md) | design tokens, rendimiento, dark mode, stack Astro + Tailwind v4 |

## Guidelines by Context

Reglas de contexto en `rules/`:

- `rules/tailwind-css-first.rule.md` → Configuración CSS-first y design tokens
- `rules/tailwind-utilities.rule.md` → Utilidades, variantes y buenas prácticas de estilado

## Critical Rules

### MUST DO

- Usar `@import "tailwindcss";` como punto de entrada (v4 CSS-first)
- Configurar design tokens con `@theme` en vez de inline arbitrary values repetidos
- Usar `@layer base/components/utilities` para organizar CSS propio
- Usar `@apply` para reutilizar utilidades dentro de componentes CSS
- Preferir tokens de `@theme` sobre arbitrary values (`bg-mint-500` en vez de `bg-[#...]`)
- Usar variantes modernas: `dark:`, `sm:`/`md:`/`lg:`, `@container`, `:user-valid`
- En Astro, instalar con `@tailwindcss/vite` y añadir el plugin en `astro.config.mjs`
- Verificar APIs dudosas en tailwindcss.com o vía Context7
- Usar el MCP 'Tailwindcss' para convertir CSS→utilidades, generar paletas y plantillas

### MUST NOT DO

- No crear `tailwind.config.js` en v4 — la configuración es CSS-first
- No usar `@tailwind base/components/utilities` (v3) — en v4 es `@import "tailwindcss"`
- No abusar de arbitrary values (`[#...]`) cuando existe un token en `@theme`
- No inventar directivas o clases sin verificar en tailwindcss.com
- No usar `@apply` con clases que dependen de variantes (no funciona en v4)
- No olvidar añadir el plugin `@tailwindcss/vite` en `astro.config.mjs`/`vite.config`
- No confundir `@tailwindcss/postcss` (Next.js) con `@tailwindcss/vite` (Vite/Astro/Nuxt)

## Quick Reference

### Instalación (resumen)

```bash
# Vite / Astro / Nuxt
npm i tailwindcss @tailwindcss/vite

# Next.js
npm i tailwindcss @tailwindcss/postcss postcss

# Standalone (Node/Express/Hono, sin bundler)
npm i -D tailwindcss @tailwindcss/cli
```

> **Versión:** Tailwind v4 estable. Verifica la versión del proyecto vía Context7 antes de dar código.

### Configuración CSS-first mínima

```css
@import "tailwindcss";

@theme {
  --color-mint-500: oklch(0.72 0.11 178);
  --font-display: "Satoshi", sans-serif;
  --breakpoint-3xl: 120rem;
}
```

### Ejemplo de componente (Astro + Tailwind v4)

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

## Output Format

Al implementar estilado con Tailwind v4, proporciona:

1. Método de instalación por framework (paquete + configuración)
2. `@import "tailwindcss"` + directivas CSS-first usadas
3. Design tokens definidos en `@theme` (si aplica)
4. Utilidades y variantes aplicadas (responsive, dark, container)
5. Plugins registrados (si aplica)
6. Notas de rendimiento (tree-shaking, evitar arbitrary values)

## Technologies

Tailwind CSS v4 (CSS-first), `@tailwindcss/vite`, `@tailwindcss/postcss`, `@tailwindcss/cli`, `@tailwindcss/typography`, `@tailwindcss/forms`, `@tailwindcss/container-queries`, Vite, Astro, Next.js, Nuxt, Node/Express/Hono, oklch, design tokens, container queries, dark mode, MCP 'Tailwindcss', MCP 'Context7'