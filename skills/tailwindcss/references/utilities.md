# Utilidades y variantes (Tailwind v4)

Guía de utilidades core, variantes modernas, container queries, dark mode y responsive en Tailwind v4. Referencia: https://tailwindcss.com/docs/utility-first

## Utilidades core

### Layout (flex / grid)

```html
<div class="flex items-center justify-between gap-4">
  <div class="grid grid-cols-3 gap-4">
    <div class="col-span-2">...</div>
    <div>...</div>
  </div>
</div>
```

- Flex: `flex`, `flex-col`, `items-center`, `justify-between`, `gap-*`, `flex-1`
- Grid: `grid`, `grid-cols-*`, `col-span-*`, `row-span-*`, `gap-*`
- Spacing: `p-*`, `m-*`, `px-*`, `py-*`, `mt-*`, `space-x-*`, `space-y-*`

### Colores (desde @theme)

```html
<div class="bg-mint-500 text-white hover:bg-mint-600">
  <span class="text-slate-600 dark:text-slate-300">...</span>
</div>
```

- `bg-*`, `text-*`, `border-*`, `fill-*`, `stroke-*`
- Opacidad: `bg-mint-500/50`, `text-white/80`

### Tipografía

```html
<p class="font-display text-xl font-bold tracking-tight leading-relaxed">
  Texto
</p>
```

- `font-*` (familia), `text-*` (tamaño), `font-bold` (peso), `tracking-*`, `leading-*`

---

## Responsive (breakpoints)

```html
<div class="flex flex-col md:flex-row lg:justify-between">
  <div class="w-full md:w-1/2">...</div>
</div>
```

- `sm:` (640px), `md:` (768px), `lg:` (1024px), `xl:` (1280px), `2xl:` (1536px)
- Breakpoints custom: `--breakpoint-3xl` en `@theme` → `3xl:`

---

## Dark mode

```html
<div class="bg-white dark:bg-slate-800">
  <p class="text-slate-900 dark:text-white">...</p>
</div>
```

- Por defecto `dark:` usa `prefers-color-scheme` (media query)
- Class-based: `@custom-variant dark (&:where(.dark, .dark *))` + clase `.dark` en `<html>`

---

## Container queries

```html
<div class="@container">
  <div class="flex flex-col @sm:flex-row @8xl:flex-row">
    <!-- ... -->
  </div>
</div>
```

- `@container` en el padre activa el container query
- Variantes: `@sm:`, `@md:`, `@lg:`, `@xl:`, `@8xl:` (según el ancho del contenedor, no del viewport)
- Requiere el plugin `@tailwindcss/container-queries` (o está integrado en v4)

---

## Variantes de estado modernas

```html
<input class="border ... valid:border-green-500 invalid:border-red-500
              :user-valid:border-green-500 :user-invalid:border-red-500" />
```

- `:user-valid` / `:user-invalid` — solo tras interacción del usuario (mejor que `valid:`/`invalid:`)
- `:placeholder-shown` — cuando el placeholder está visible
- `has-*` — `has-[:checked]`, `has-[img]` (basado en `:has()`)
- `hover:`, `focus:`, `focus-visible:`, `active:`, `disabled:`, `checked:`

---

## Arbitrary values (usar con moderación)

```html
<div class="w-[200px] bg-[#123456] text-[13px]">
  ...
</div>
```

> Preferir tokens de `@theme` sobre arbitrary values repetidos. Los arbitrary values no se tree-shakean tan eficientemente y dificultan el mantenimiento.

---

## Notas

- Verifica clases/directivas dudosas en tailwindcss.com o vía Context7.
- Usa el MCP 'Tailwindcss' para convertir CSS→utilidades, generar paletas y plantillas.