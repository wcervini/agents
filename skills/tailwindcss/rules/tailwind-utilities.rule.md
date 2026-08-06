---
description: Reglas de utilidades, variantes y buenas prácticas de estilado con Tailwind v4
globs:
  - "**/*.astro"
  - "**/*.html"
  - "**/*.vue"
  - "**/*.jsx"
  - "**/*.tsx"
---

# Tailwind Utilities Rules

## MUST DO

- Usar variantes modernas: `dark:`, `sm:`/`md:`/`lg:`, `@container`, `:user-valid`
- Usar container queries (`@container` + `@sm:`/`@8xl:`) para componentes reutilizables
- Usar `:user-valid`/`:user-invalid` para estados de formulario tras interacción
- Preferir tokens de `@theme` sobre arbitrary values
- Usar el MCP 'Tailwindcss' para convertir CSS→utilidades, generar paletas y plantillas

## MUST NOT DO

- No abusar de arbitrary values (`bg-[#...]`, `w-[200px]`) cuando existe un token
- No usar `valid:`/`invalid:` si `:user-valid`/`:user-invalid` aplica (mejor UX)
- No inventar clases sin verificar en tailwindcss.com o vía Context7

## Patrones

### Responsive + dark mode

```html
<div class="flex flex-col md:flex-row bg-white dark:bg-slate-800">
  <p class="text-slate-900 dark:text-white">...</p>
</div>
```

### Container queries

```html
<div class="@container">
  <div class="flex flex-col @sm:flex-row @8xl:flex-row">
    <!-- ... -->
  </div>
</div>
```

### Estado moderno de formulario

```html
<input class=":user-valid:border-green-500 :user-invalid:border-red-500" />
```

### Componente Astro con Tailwind

```astro
<div class="rounded-2xl bg-white p-6 shadow-sm dark:bg-slate-800">
  <h2 class="font-display text-xl font-bold">Hola, Tailwind v4</h2>
  <button class="mt-4 rounded-lg bg-mint-500 px-4 py-2 text-white hover:bg-mint-600">
    Acción
  </button>
</div>
```