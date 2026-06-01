---
name: local-docs-context
description: |
  Lee documentación local de Astro (español) y Tailwind CSS desde el disco para responder consultas con información actualizada offline.

  Activar cuando el usuario pregunte sobre:
  - Astro, componentes Astro, islands, content collections, directivas, enrutamiento, SSR, adaptadores, view transitions, sesiones, actions, middleware, i18n, `astro:env`, `server:defer`
  - Tailwind CSS, utilidades, diseño responsive, variantes, personalización del theme, `tailwind.config`, `@apply`, `@layer`, `@theme`, `dark mode`, `container queries`
  - Cualquier concepto, API, configuración o guía relacionada con Astro o Tailwind CSS

  NO activar para:
  - Next.js, Remix, SvelteKit u otros frameworks no relacionados
  - CSS genérico sin relación con Tailwind
---

# Documentación Local de Astro (ES) y Tailwind CSS

## Ubicaciones

### Astro Docs (Español)
```
C:\Users\Underghround\Desktop\astro-docs\src\content\docs\es\
├── basics/           → conceptos fundamentales
├── concepts/         → why-astro, islands, hydration, routing, ssg, ssr
├── guides/           → actions, images, styling, sessions, view-transitions, etc.
├── recipes/          → ejemplos prácticos
├── reference/        → API reference, directivas, sintaxis
├── tutorial/         → tutorial paso a paso
├── getting-started.mdx
├── install-and-setup.mdx
├── upgrade-astro.mdx
├── editor-setup.mdx
├── develop-and-build.mdx
├── astro-courses.mdx
└── contribute.mdx
```

### Tailwind CSS Docs
```
C:\Users\Underghround\Desktop\tailwindcss.com\src\docs\
├── img/               → imágenes
├── utils/             → utilidades (colores, etc.)
├── [197+ archivos .mdx] → cada utility class, guía y referencia
```

## Flujo de trabajo

1. **Detecta el tema** de la consulta del usuario (Astro o Tailwind CSS)
2. **Identifica el archivo relevante** dentro de la estructura de documentación local usando `glob` y `grep`
3. **Lee el archivo** con la herramienta `read` para obtener el contenido completo
4. **Usa el contenido** para responder la consulta del usuario basándote en la documentación local

## Estrategia de búsqueda

Para encontrar el archivo adecuado dentro de la documentación local:

### Astro
```powershell
# Buscar por palabra clave en la documentación de Astro
Get-ChildItem -Path "C:\Users\Underghround\Desktop\astro-docs\src\content\docs\es" -Recurse -Filter "*.mdx" | Select-String -Pattern "<término>"
```

### Tailwind CSS
```powershell
# Buscar por palabra clave en la documentación de Tailwind
Get-ChildItem -Path "C:\Users\Underghround\Desktop\tailwindcss.com\src\docs" -Filter "*.mdx" | Select-String -Pattern "<término>"
```

## Reglas

- **Siempre** consulta la documentación local antes de responder preguntas sobre Astro o Tailwind CSS
- **No inventes** APIs, configuraciones o sintaxis que no aparezcan en la documentación local
- **Prioriza** la documentación local sobre el conocimiento general si hay conflicto
- **Menciona la fuente** al citar información (ej: "según la documentación local de Astro en español...")
- Si el archivo relevante es muy extenso, usa `offset` y `limit` para leerlo por partes
- Los archivos MDX pueden contener componentes React (`import` de componentes); enfócate en el contenido markdown/textual

## Temas principales cubiertos

### Astro (Español)
- Instalación, configuración, CLI
- Islands architecture, directivas `client:*`, `server:defer`
- Content Collections / Content Layer API
- Enrutamiento (estático, dinámico, endpoints)
- SSR, adapters (Node, Vercel, Netlify, Cloudflare, Deno)
- View Transitions, Sessiones, Actions, Middleware
- `astro:env`, i18n, imágenes, estilos
- Referencia de sintaxis, directivas, config

### Tailwind CSS
- Utility classes (flex, grid, spacing, typography, colors, etc.)
- Responsive design (`sm:`, `md:`, `lg:`, etc.)
- Dark mode, estado (`hover:`, `focus:`, `active:`, etc.)
- Personalización con `tailwind.config.js`/`tailwind.config.ts`
- `@theme`, `@apply`, `@layer`, `@config`
- Container queries, animaciones, filters
- Plugins, funciones y presets
