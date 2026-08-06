# Zod Cheatsheet

**Version:** 1.0.0 | **Zod 4.x** | **Updated:** 2026-08-06 | **Author**: delineas

---

## Quick Reference Card

| Tema | Regla práctica |
|------|---------------|
| **Instalación** | `npm i zod` · `pnpm add zod` · `bun add zod` — es librería, NO CLI |
| **Primitivas** | `z.string()`, `z.number()`, `z.boolean()`, `z.date()`, `z.literal('x')`, `z.enum(['a','b'])`, `z.email()`, `z.uuid()` |
| **Composición** | `z.object({...})`, `z.array(t)`, `z.union([a,b])`, `z.discriminatedUnion('kind', [...])`, `z.tuple([a,b])`, `z.record(k, v)`, `z.intersection(a, b)`, `z.lazy(() => ...)` |
| **Modificadores** | `.optional()`, `.nullable()`, `.default(v)`, `.catch(v)`, `.readonly()`, `.brand()` |
| **Parseo** | `.parse()` lanza; `.safeParse()` devuelve `{ success, data\|error }`; async: `.parseAsync()`/`.safeParseAsync()` |
| **Refinamiento** | `.refine(fn, msg)`, `.superRefine((v, ctx) => ctx.addIssue(...))`, `.transform(fn)`, `z.pipe(a, b, c)` |
| **Coerción** | `z.coerce.number()`, `z.coerce.date()`, `z.coerce.boolean()` — input: `unknown` |
| **Tipos** | `z.infer<typeof S>` = output; `z.input<typeof S>` = entrada (pre-transform); `z.output` = alias de infer |
| **Errores** | `ZodError` con `.issues[]` (`code`, `path`, `message`); v4: `z.treeifyError(err)`, `z.flattenError(err)` (`.format()`/`.flatten()` deprecados) |
| **errorMap** | `schema.parse(v, { errorMap })` o `z.setErrorMap(customMap)` global; `z.ZodIssueCode.*` |
| **JSON Schema** | `z.toJSONSchema(schema, { target: 'draft-2020-12' })` (v4, experimental/estable) |
| **Astro** | `import { z } from 'astro/zod'` — NUNCA del paquete `zod` (Content Layer, Actions, astro:env) |
| **React Hook Form** | `import { zodResolver } from '@hookform/resolvers/zod'`; `useForm({ resolver: zodResolver(schema) })` |
| **tRPC** | Schemas Zod directamente como `input`/`output` de procedimientos |
| **Express/Fastify** | Validar `req.body`/payload con `schema.safeParse` en el borde de la API |
| **Ref. raíz** | https://zod.dev/ — verificar cualquier API dudosa |

---

## Decision Trees

### Estrategia de parseo
```
¿El dato debe ser siempre válido (invariante)?
├── Sí → .parse(data)              // lanza ZodError
└── No → .safeParse(data)          // { success: true, data } | { success: false, error }
         └── ¿Validación async? → .parseAsync() / .safeParseAsync()
```

### unión discriminada vs union
```
¿Los objetos comparten un campo literal discriminador (type/kind/status)?
├── Sí → z.discriminatedUnion('kind', [SchemaA, SchemaB])   // errores claros, rápido
└── No → z.union([SchemaA, SchemaB])                        // prueba en orden
```

### coerce vs transform
```
¿Solo convertir el tipo en runtime (string→number, string→date)?
├── Sí → z.coerce.number() / z.coerce.date()        // input: unknown
└── ¿Además transformar el valor (trim, slug, parsear)? → .transform(fn)
```

---

## Critical "NEVER Do" List

- **NEVER** usar `any` — usa `unknown` y deja que el esquema estreche el tipo
- **NEVER** duplicar interfaces manuales si `z.infer<typeof schema>` ya genera el tipo
- **NEVER** usar `.parse()` en APIs/forms — usa `safeParse` para controlar el fallo
- **NEVER** inventar métodos de Zod — verifica en zod.dev o Context7
- **NEVER** importar `z` de `zod` en proyectos Astro — usa `astro/zod`
- **NEVER** usar `.format()`/`.flatten()` en Zod v4 — deprecados; usa `z.treeifyError`/`z.flattenError`
- **NEVER** validar solo en el cliente — valida siempre en el borde del servidor
- **NEVER** usar `z.date()` para fechas que llegan como string — usa `z.coerce.date()`

---

## Zod v4 Notes (Post-Training / Verificado con Context7)

### Errores: funciones standalone (v4)

En Zod 4 los métodos de formato de errores del `ZodError` están **deprecados**:

```typescript
// ✅ v4 (recomendado)
z.treeifyError(error);   // estructura anidada que refleja el esquema
z.flattenError(error);   // { formErrors: string[], fieldErrors: { [campo]: string[] } }
z.formatError(error);    // formato anidado estilo v3 .format()

// ❌ v3 / deprecado en v4
error.format();
error.flatten();
error.addIssue();
```

- `treeifyError` es útil para estructuras anidadas; `flattenError` para la mayoría de esquemas planos.
- `ZodIssue` conserva `code`, `path: (string|number)[]`, `message`.
- `ZodIssueCode`: `invalid_type`, `invalid_string`, `too_big`, `too_small`, `custom`, `unrecognized_keys`, `invalid_union`, `invalid_enum_value`, etc.

### z.toJSONSchema (v4) — OpenAPI / JSON Schema

```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string(),
  age: z.number().int(),
});

const jsonSchema = z.toJSONSchema(schema, { target: 'draft-2020-12' });
```

- Alternativa para OpenAPI: paquete `zod-openapi` (`@samchungy/zod-openapi`) con `.openapi({ ... })`.
- Para generar docs OpenAPI v3.1 completa, la comunidad usa `zod-openapi` o `@asteasolutions/zod-to-openapi`.

### z.record

```typescript
const Scores = z.record(z.string(), z.number());
// v4: valida claves y valores; modo "strict" por defecto (claves no reconocidas → error)
const Loose = z.record(z.string(), z.number(), { mode: 'loose' }); // pasa claves inválidas
```

### z.record con claves numéricas

```typescript
const ById = z.record(z.number(), z.string()); // acepta { "1": "a" } (fallback numérico en claves string)
```

### z.lazy — recursión

```typescript
const baseCategory = z.object({ name: z.string() });
type Category = z.infer<typeof baseCategory> & { subcategories: Category[] };
const categorySchema: z.ZodType<Category> = baseCategory.extend({
  subcategories: z.lazy(() => categorySchema.array()),
});
```

### z.coerce — tipos de entrada

```typescript
const A = z.coerce.number();
type AInput = z.input<typeof A>;   // unknown (acepta "42" o 42)
const B = z.coerce.number<number>();
type BInput = z.input<typeof B>;   // number
```

### Zod mini (bundle pequeño)

```typescript
import { z } from 'zod/mini'; // variante ligera de Zod 4, sin algunos features
```

---

## Integraciones (mínimas)

| Integración | Paquete | Ejemplo mínimo |
|-------------|---------|----------------|
| Astro (Content Layer / Actions / env) | `astro/zod` (built-in) | `import { z } from 'astro/zod'` → `schema: z.object({...})` |
| React Hook Form | `@hookform/resolvers/zod` | `useForm({ resolver: zodResolver(schema) })` |
| tRPC | `zod` (directo) | `input: z.object({ id: z.string() })` |
| Express | `zod` + validación manual | `const r = schema.safeParse(req.body); if (!r.success) return 400` |
| Fastify | `zod` + plugin | `fastify.post('/x', { schema: { body: schema } })` (con `fastify-type-provider-zod`) |
| OpenAPI/JSON Schema | `zod` / `zod-openapi` | `z.toJSONSchema(schema)` o `.openapi({...})` |

---

## Key Patterns (Brief)

- **`z.object()`** — todas las propiedades requeridas por defecto; usa `.optional()`/`.default()` explícito
- **`.extend()`** — ampliar un esquema base: `base.extend({ campo: z.string() })`
- **`.pick()`/`.omit()`** — `User.pick({ id: true })`, `User.omit({ password: true })`
- **`.partial()`** — todas las claves opcionales (útil para PATCH)
- **`.array()`** — `z.array(z.string()).min(1).max(10)`
- **`z.pipe()`** — `z.pipe(z.string(), z.coerce.number())` (encadenar validaciones/transforms)
- **`.catch()`** — valor de respaldo ante error: `z.string().catch('default')`
- **`.default()`** — valor por defecto solo si el input es `undefined`
- **`.brand()`** — tipos nominales: `z.string().brand<'UserId'>()`

## References

Docs a detalle con ejemplos completos en `references/`:

`parsing.md` `type-inference.md` `errors.md` `integrations.md`
