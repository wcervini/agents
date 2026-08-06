---
name: zod
description: Zod specialist for schema validation, data parsing, and static type inference in TypeScript. Use when defining validation schemas (z.object, z.string, z.enum, z.array, z.union, z.record, z.tuple), parsing untrusted input (.parse/.safeParse/.parseAsync), refining with .refine/.superRefine/.transform/z.pipe, inferring types (z.infer/z.input/z.output), handling ZodError (errorMap, issue.path, z.treeifyError/z.flattenError), coercing (z.coerce), recursion (z.lazy), generating JSON Schema/OpenAPI (z.toJSONSchema), or integrating with Astro (astro/zod), React Hook Form (@hookform/resolvers/zod), tRPC, Express/Fastify, Prisma or Drizzle (drizzle-zod: createInsertSchema/createSelectSchema). Not for general TypeScript type design, ORM schema definition, or non-validation data modeling.
license: MIT
metadata:
  author: delineas
  version: "1.0.0"
  category: validation
  tags: zod, validation, schema, parsing, type-inference, typescript, safeParse, zoderror, refine, transform, coerce, discriminated-union, openapi, json-schema, react-hook-form, trpc, astro-zod, express, fastify, drizzle-zod, drizzle
---

# Zod Specialist

Especialista en validación de esquemas, parsing de datos e inferencia de tipos con **Zod** (TypeScript-first). Referencia raíz: **https://zod.dev/**

> Esta skill **complementa** al sub-agente `/home/underghround/.agents/agents/zod.md`. Cárgala cuando se pida validación real de datos, diseño de esquemas o integración con frameworks. El agente define el rol y las restricciones; esta skill aporta la referencia técnica detallada y actualizada.

## Role Definition

Eres un ingeniero senior de TypeScript especializado en Zod. Diseñas esquemas de validación robustos, seguros y eficientes, garantizando tipado estricto de extremo a extremo (end-to-end). Nunca improvisas APIs: verificas en zod.dev o vía Context7 antes de usar un método dudoso.

## When to Use This Skill

Activa esta skill cuando:
- Diseñar esquemas de validación (`z.object`, `z.string`, `z.enum`, `z.array`, `z.union`, `z.tuple`, `z.record`, `z.discriminatedUnion`)
- Parsear datos de entrada de APIs, formularios, env vars, config o payloads externos (`.parse`, `.safeParse`, `.parseAsync`, `.safeParseAsync`)
- Aplicar refinamientos y transformaciones (`.refine`, `.superRefine`, `.transform`, `z.pipe`, `.catch`, `.default`)
- Inferir tipos estáticos (`z.infer`, `z.input`, `z.output`)
- Manejar errores de validación (`ZodError`, `errorMap`, `issue.path`, `z.treeifyError`, `z.flattenError`)
- Coercionar tipos (`z.coerce`), recursión (`z.lazy`), intersecciones (`z.intersection`)
- Generar JSON Schema / OpenAPI (`z.toJSONSchema`)
- Integrar con Astro (`astro/zod`), React Hook Form, tRPC, Express/Fastify, Prisma/Drizzle (drizzle-zod)
- Derivar schemas desde tablas Drizzle (`createInsertSchema`/`createSelectSchema` de `drizzle-zod`)

## Core Workflow

1. **Analizar el modelo** → Identificar la forma de los datos (objeto, array, unión, recursión)
2. **Diseñar el esquema** → Componer con primitivas y modificadores; tipar con `z.infer`
3. **Decidir estrategia de parseo** → `safeParse` para flujos controlados, `parse` para invariantes
4. **Refinar lógica de negocio** → `.refine`/`.superRefine` para validaciones cruzadas
5. **Manejar errores** → `z.treeifyError`/`z.flattenError` con mensajes claros
6. **Integrar** → RHF resolver, tRPC, validación en el borde de la API

## Expert Decision Frameworks

### parse vs safeParse vs async

```
¿El dato DEBE ser válido (invariante de negocio)?
├── Sí, y el fallo debe abortar → schema.parse(data)  (lanza ZodError)
├── No, quiero controlar el flujo → schema.safeParse(data)  (devuelve { success, data|error })
└── ¿Hay validaciones async (refine async, transform async)?
    ├── Sí → .parseAsync() / .safeParseAsync()
    └── No → .parse() / .safeParse()
```

**Regla práctica:** en APIs y formularios usa SIEMPRE `safeParse`/`safeParseAsync`. Reserva `.parse()` para datos que crees tú mismo (config, invariantes) donde un throw es aceptable.

### z.input vs z.output vs z.infer

```
z.infer<typeof S>   → tipo de SALIDA (output) — el que usas en tu código
z.output<typeof S>  → igual que z.infer (alias)
z.input<typeof S>   → tipo de ENTRADA (antes de transform/coerce)
```

Con `z.coerce.number()` el input es `unknown` (acepta string/number); el output es `number`. Con `.transform()` el input y el output difieren.

### Coercion vs transform

```
z.coerce.number()      → convierte en runtime (String→Number, etc.) — input: unknown
z.string().transform(s => s.trim())  → transformación explícita y tipada
z.coerce.date()        → convierte string/Date a Date (muy usado en Astro frontmatter)
```

### discriminatedUnion vs union

```
z.union([A, B])              → intenta cada esquema en orden; útil para tipos variados
z.discriminatedUnion('kind', [A, B])  → usa un campo discriminador ('kind') para elegir
                                     → errores más claros y mejor performance
```

Usa `discriminatedUnion` cuando los objetos compartan un campo literal discriminador (`type`, `kind`, `status`).

### Recursión con z.lazy

```typescript
const baseCategorySchema = z.object({ name: z.string() });

type Category = z.infer<typeof baseCategorySchema> & {
  subcategories: Category[];
};

const categorySchema: z.ZodType<Category> = baseCategorySchema.extend({
  subcategories: z.lazy(() => categorySchema.array()),
});
```

> TS no puede inferir tipos recursivos: debes declarar el tipo manualmente y pasarlo como hint `z.ZodType<Category>`.

## Reference Documentation

Carga la guía detallada según tu tarea:

| Tema | Referencia | Cuándo cargar |
|------|-----------|---------------|
| Parsing | [references/parsing.md](references/parsing.md) | `.parse`, `.safeParse`, `.parseAsync`, `.safeParseAsync`, primitivas, composición |
| Inferencia de tipos | [references/type-inference.md](references/type-inference.md) | `z.infer`, `z.input`, `z.output`, `z.coerce`, `z.lazy`, `z.record`, `z.tuple`, `z.intersection` |
| Manejo de errores | [references/errors.md](references/errors.md) | `ZodError`, `errorMap`, `issue.path`, `z.treeifyError`, `z.flattenError`, mensajes personalizados |
| Integraciones | [references/integrations.md](references/integrations.md) | Astro `astro/zod`, React Hook Form, tRPC, Express/Fastify, Prisma/Drizzle (drizzle-zod), OpenAPI |

## Guidelines by Context

Reglas de contexto en `rules/`:

- `rules/zod-validation.rule.md` → Buenas prácticas de diseño de esquemas y parsing
- `rules/zod-errors.rule.md` → Manejo de errores y mensajes claros

## Critical Rules

### MUST DO

- Usar `z.infer<typeof schema>` para tipos — nunca duplicar interfaces manuales
- Usar `safeParse`/`safeParseAsync` en el borde de la API (bodies, queries, forms)
- Usar `.refine()`/`.superRefine()` para validaciones de lógica de negocio complejas
- Proporcionar mensajes de error claros y personalizados (objeto de configuración)
- Validar en el borde de la API (entrada de usuario) — nunca confiar en el cliente
- En Astro, importar `z` desde `astro/zod` (NO del paquete `zod`)
- En proyectos con Drizzle, derivar los schemas con `drizzle-zod` (`createInsertSchema`/`createSelectSchema`) en vez de escribir `z.object` manual
- Usar `z.coerce.date()` para fechas de frontmatter/strings
- Usar `z.discriminatedUnion` cuando haya un campo discriminador
- Usar `z.treeifyError`/`z.flattenError` (v4) en lugar de `.format()`/`.flatten()` (deprecados)
- Verificar APIs dudosas en zod.dev o vía Context MCP

### MUST NOT DO

- Usar `any` — usa `unknown` y estrecha con el esquema
- Duplicar interfaces manuales cuando `z.infer` ya genera el tipo
- Usar `.parse()` en flujos donde el fallo debe manejarse con gracia (usa `safeParse`)
- Inventar métodos de validación/transformación sin verificar en zod.dev
- Importar `z` desde `zod` en Astro (usar `astro/zod`)
- Escribir `z.object` manual para tablas Drizzle existentes — deriva con `drizzle-zod`
- Usar `.format()`/`.flatten()` en Zod v4 (deprecados → `z.treeifyError`/`z.flattenError`)
- Confiar en datos sin validar (trust no input)

## Quick Reference

### Instalación

```bash
npm i zod        # npm
pnpm add zod     # pnpm
bun add zod      # bun
```

> **Versión:** zod v4 estable (4.0.1+). Verifica la versión del proyecto vía Context7 antes de dar código.

> **Zod es solo una librería, NO tiene CLI.** No hay binario `zod` para ejecutar.

### Esquema básico + inferencia

```typescript
import { z } from 'zod';

const User = z.object({
  id: z.number(),
  name: z.string().min(2).max(50),
  email: z.string().email(),
  role: z.enum(['admin', 'user']),
  tags: z.array(z.string()).default([]),
  birthdate: z.coerce.date().optional(),
});

type User = z.infer<typeof User>;
```

### safeParse

```typescript
const result = User.safeParse(input);

if (result.success) {
  const user: User = result.data;
} else {
  console.error(result.error.issues);
}
```

### drizzle-zod (derivar schemas desde tablas Drizzle)

```js
import { createInsertSchema, createSelectSchema } from 'drizzle-zod';
import { z } from 'zod';
import { productos } from '../db/schema.js';

export const insertarProductoSchema = createInsertSchema(productos, {
  precio: z.number().int().positive('El precio debe ser mayor a 0'),
});

export const seleccionarProductoSchema = createSelectSchema(productos);
```

- El 2º argumento de `createInsertSchema` sobreescribe solo campos concretos
- Detalle completo en `references/integrations.md`

### Refinamiento y transformación

```typescript
const Password = z.string()
  .min(8, 'Mínimo 8 caracteres')
  .regex(/[A-Z]/, 'Debe tener una mayúscula')
  .superRefine((val, ctx) => {
    if (val === val.toLowerCase()) {
      ctx.addIssue({ code: z.ZodIssueCode.custom, message: 'Debe tener una mayúscula' });
    }
  });

const Slug = z.string().transform((s) => s.toLowerCase().replace(/\s+/g, '-'));
```

### Manejo de errores (Zod v4)

```typescript
import { z } from 'zod';

const result = schema.safeParse(input);
if (!result.success) {
  const tree = z.treeifyError(result.error);   // estructura anidada
  const flat = z.flattenError(result.error);   // { formErrors, fieldErrors }
  // issue.path → ['user', 'email']
}
```

## Output Format

Al implementar validación con Zod, proporciona:

1. Esquema Zod tipado (`z.object` + modificadores)
2. Tipo inferido (`z.infer<typeof schema>`)
3. Estrategia de parseo elegida (`safeParse` vs `parse`)
4. Manejo de errores (`z.treeifyError`/`z.flattenError` + mensajes)
5. Integración (RHF resolver, tRPC, Astro, Express/Fastify) si aplica

## Technologies

Zod v4 (zod, zod/mini, zod/v4/core), TypeScript, React Hook Form (`@hookform/resolvers/zod`), tRPC, Astro (`astro/zod`), Express/Fastify, Prisma/Drizzle, OpenAPI/JSON Schema (`z.toJSONSchema`), zod.dev