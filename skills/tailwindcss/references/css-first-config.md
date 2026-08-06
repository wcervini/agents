# Configuración CSS-first (Tailwind v4)

Guía de la configuración **CSS-first** de Tailwind v4. En v4 **no existe `tailwind.config.js`**: toda la configuración vive en el CSS mediante directivas. Referencia: https://tailwindcss.com/docs/theme

## Punto de entrada

```css
@import "tailwindcss";
```

Reemplaza a las directivas `@tailwind base/components/utilities` de v3.

---

## @theme — design tokens

`@theme` define design tokens que Tailwind convierte automáticamente en **utilidades** y **custom properties** CSS.

```css
@import "tailwindcss";

@theme {
  /* Colores → bg-*, text-*, border-*, etc. */
  --color-mint-500: oklch(0.72 0.11 178);
  --color-avocado-100: oklch(0.99 0 0);
  --color-avocado-500: oklch(0.84 0.18 117.33);
  --color-avocado-600: oklch(0.53 0.12 118.34);

  /* Fuentes → font-* */
  --font-display: "Satoshi", "sans-serif";

  /* Breakpoints → variantes responsive (3xl:) */
  --breakpoint-3xl: 1920px;

  /* Easing → ease-* */
  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
  --ease-snappy: cubic-bezier(0.2, 0, 0, 1);
}
```

### Convenciones de nombres

| Prefijo | Genera utilidades | Ejemplo |
|---------|-------------------|---------|
| `--color-*` | `bg-*`, `text-*`, `border-*`, `fill-*`, `stroke-*` | `--color-mint-500` → `bg-mint-500` |
| `--font-*` | `font-*` | `--font-display` → `font-display` |
| `--breakpoint-*` | variantes responsive | `--breakpoint-3xl` → `3xl:` |
| `--ease-*` | `ease-*` | `--ease-snappy` → `ease-snappy` |
| `--shadow-*` | `shadow-*` | `--shadow-sm` → `shadow-sm` |

> Los tokens también se exponen como custom properties CSS: `var(--color-mint-500)`.

### Breakpoints custom

```css
@theme {
  --breakpoint-xs: 30rem;
  --breakpoint-2xl: 100rem;
  --breakpoint-3xl: 120rem;
}
```

---

## @layer — organizar CSS propio

```css
@layer base {
  /* Reset y estilos globales */
  body {
    @apply bg-slate-50 text-slate-900;
  }
}

@layer components {
  /* Componentes reutilizables */
  .btn {
    @apply rounded-lg px-4 py-2 font-semibold;
  }
}

@layer utilities {
  /* Utilidades propias */
  .text-balance {
    text-wrap: balance;
  }
}
```

---

## @apply — reutilizar utilidades

```css
.btn-primary {
  @apply rounded-lg bg-mint-500 px-4 py-2 text-white hover:bg-mint-600;
}
```

> ⚠️ En v4, `@apply` **no funciona con clases que dependen de variantes** (p. ej. `dark:bg-slate-800` dentro de `@apply`). Usa `@custom-variant` o aplica la variante en el markup.

---

## @variant y @custom-variant

### @variant — definir una variante nueva

```css
@variant pointer-coarse {
  @media (pointer: coarse) {
    @slot;
  }
}
```

### @custom-variant — dark mode class-based

```css
@custom-variant dark (&:where(.dark, .dark *));
```

---

## Notas

- **v4 es CSS-first**: no hay `tailwind.config.js`. Toda la configuración vive en el CSS.
- `@theme` genera utilidades **y** custom properties automáticamente.
- Verifica directivas dudosas en tailwindcss.com o vía Context7.