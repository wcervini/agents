# Plugins oficiales (Tailwind v4)

Guía de los plugins oficiales de Tailwind CSS v4 y cómo registrarlos. Referencia: https://tailwindcss.com/docs/plugins

## Plugins oficiales

| Plugin | Paquete | Utilidades que aporta |
|--------|---------|----------------------|
| Typography | `@tailwindcss/typography` | `prose`, `prose-lg`, `prose-invert` (estilos de contenido rico) |
| Forms | `@tailwindcss/forms` | reset y estilos base para inputs, selects, textareas |
| Container Queries | `@tailwindcss/container-queries` | `@container`, `@sm:`, `@md:`, `@8xl:` |
| PostCSS (Next) | `@tailwindcss/postcss` | integración con Next.js vía PostCSS |

> En v4, **container queries** y **aspect-ratio** están integrados en el core (no requieren plugin separado). El plugin `@tailwindcss/container-queries` sigue disponible para compatibilidad.

---

## Instalación

```bash
# Typography
npm i @tailwindcss/typography

# Forms
npm i @tailwindcss/forms

# Container queries (si no está integrado)
npm i @tailwindcss/container-queries
```

---

## Registro en v4 (CSS-first)

En v4 con Vite/Astro/Nuxt, los plugins se registran **vía CSS** con la directiva `@plugin`:

```css
@import "tailwindcss";

@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";
```

### Uso de typography

```html
<article class="prose prose-lg dark:prose-invert">
  <h1>Título</h1>
  <p>Contenido con estilos tipográficos automáticos.</p>
</article>
```

### Uso de forms

```html
<input class="rounded-md border-gray-300 focus:ring-2 focus:ring-mint-500" />
```

---

## Registro en CLI standalone (tailwind.config.js)

> Solo aplica al CLI standalone (`@tailwindcss/cli`), que aún admite `tailwind.config.js` para plugins:

```js
// tailwind.config.js
module.exports = {
  plugins: [require("@tailwindcss/forms"), require("@tailwindcss/typography")],
};
```

---

## Next.js: @tailwindcss/postcss

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

## Notas

- En v4, **container queries y aspect-ratio están integrados** en el core.
- En Vite/Astro/Nuxt usa `@plugin` en CSS; en CLI standalone usa `tailwind.config.js`.
- Verifica la sintaxis de `@plugin` en tailwindcss.com o vía Context7.