# Integraciones de Zod

Cómo integrar Zod con frameworks y librerías del ecosistema.

## Tabla de integraciones

| Integración | Paquete | Ejemplo mínimo |
|-------------|---------|----------------|
| Astro (Content Layer / Actions / env) | `astro/zod` (built-in) | `import { z } from 'astro/zod'` → `schema: z.object({...})` |
| React Hook Form | `@hookform/resolvers/zod` | `useForm({ resolver: zodResolver(schema) })` |
| tRPC | `zod` (directo) | `input: z.object({ id: z.string() })` |
| Express | `zod` + validación manual | `schema.safeParse(req.body)` → 400 si falla |
| Fastify | `zod` + `fastify-type-provider-zod` | `schema: { body: schema }` |
| OpenAPI / JSON Schema | `zod` / `zod-openapi` | `z.toJSONSchema(schema)` o `.openapi({...})` |
| Drizzle (drizzle-zod) | `drizzle-zod` | `createInsertSchema(tabla, {...})` / `createSelectSchema(tabla)` — derivar schemas de la tabla |
| Prisma | `zod` (schemas de entrada) | validar payload antes de insertar |

## Astro — `astro/zod`

En Astro (Content Layer, Actions, `astro:env`) importa `z` desde `astro/zod`, **NO** del paquete `zod`:

```typescript
// src/content.config.ts
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: z.object({
    title: z.string(),
    date: z.coerce.date(),
    draft: z.boolean().default(false),
    tags: z.array(z.string()).optional(),
  }),
});

export const collections = { blog };
```

```typescript
// src/actions/index.ts
import { defineAction, z } from 'astro:actions';

export const server = {
  subscribe: defineAction({
    input: z.object({ email: z.string().email() }),
    handler: async ({ email }) => { /* ... */ },
  }),
};
```

## React Hook Form

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(2, 'Nombre muy corto'),
  email: z.string().email('Email inválido'),
});

type FormData = z.infer<typeof schema>;

function MyForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit((data) => console.log(data))}>
      <input {...register('name')} />
      {errors.name && <p>{errors.name.message}</p>}
      <input {...register('email')} />
      {errors.email && <p>{errors.email.message}</p>}
      <button type="submit">Enviar</button>
    </form>
  );
}
```

## tRPC

```typescript
import { initTRPC } from '@trpc/server';
import { z } from 'zod';

const t = initTRPC.create();

export const appRouter = t.router({
  getUser: t.procedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ input }) => {
      return await db.user.findUnique({ where: { id: input.id } });
    }),
});
```

## Express

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

const CreateUser = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

app.post('/users', (req, res) => {
  const result = CreateUser.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: z.flattenError(result.error) });
  }
  // result.data está tipado y validado
  res.status(201).json(result.data);
});
```

## Fastify (con type provider)

```typescript
import Fastify from 'fastify';
import { serializerCompiler, validatorCompiler, jsonSchemaTransform } from 'fastify-type-provider-zod';
import { z } from 'zod';

const app = Fastify();
app.setValidatorCompiler(validator);
app.setSerializerCompiler(serializerCompiler);

app.post('/users', {
  schema: {
    body: z.object({ name: z.string(), email: z.string().email() }),
  },
}, async (req) => {
  return req.body; // tipado y validado
});
```

## OpenAPI / JSON Schema

```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string(),
  age: z.number().int(),
});

const jsonSchema = z.toJSONSchema(schema, { target: 'draft-2020-12' });
```

Para OpenAPI v3.1 completa, usa `zod-openapi`:

```typescript
import { extendZodWithOpenApi } from '@asteasolutions/zod-to-openapi';
import { z } from 'zod';
extendZodWithOpenApi(z);

const schema = z.object({ name: z.string() }).openapi({ description: 'Nombre del usuario' });
```

## Drizzle — schemas zod derivados con drizzle-zod

Con Drizzle, **no escribas `z.object` a mano**: el esquema vive en Drizzle (fuente de verdad) y los schemas de validación se **derivan** de las tablas con `drizzle-zod` (`createInsertSchema`/`createSelectSchema`). Solo se sobreescriben campos concretos con refinamientos zod.

Ejemplo mínimo basado en el stack de referencia (`src/db/schema.js` + `src/schemas/producto.js`):

```js
// src/db/schema.js — fuente de verdad del esquema (sqliteTable)
import { sqliteTable, integer, text } from 'drizzle-orm/sqlite-core'

export const productos = sqliteTable('productos', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  codigo: text('codigo').notNull().unique(),
  nombre: text('nombre').notNull(),
  precio: integer('precio').notNull(),
  stock: integer('stock').notNull().default(0),
  categoria: text('categoria').notNull(),
})

// src/schemas/producto.js — schemas DERIVADOS (drizzle-zod)
import { createInsertSchema, createSelectSchema } from 'drizzle-zod'
import { z } from 'zod'
import { productos } from '../db/schema.js'

export const insertarProductoSchema = createInsertSchema(productos, {
  precio: z.number().int().positive('El precio debe ser mayor a 0'),
  stock: z.number().int().min(0, 'El stock no puede ser negativo'),
})

export const seleccionarProductoSchema = createSelectSchema(productos)
```

- `npm i drizzle-zod` (compatible con zod v4, 4.0.1+)
- `createInsertSchema(tabla, { campo: refinamiento })` → schema para INSERT; PK/auto/default quedan opcionales; el 2º argumento **sobreescribe solo campos concretos**
- `createSelectSchema(tabla)` → schema para SELECT/lectura, sin sobreescribir
- Uso típico: `input: insertarProductoSchema` en Astro actions, `schema.safeParse(body)` en APIs, `z.infer<typeof insertarProductoSchema>` para tipos
- Si la tabla usa `integer({ mode: 'boolean' })` / `integer({ mode: 'timestamp_ms' })`, drizzle-zod traduce a `z.boolean()` / `z.date()` automáticamente

## Prisma

Zod no reemplaza el schema del ORM, pero valida la entrada antes de persistir:

```typescript
import { z } from 'zod';

const CreateUser = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

const parsed = CreateUser.parse(body);
await prisma.user.create({ data: parsed });
```

## Buenas prácticas

- Valida en el borde de la API, no en el cliente
- En Astro usa `astro/zod` (evita duplicar dependencias)
- Con Drizzle deriva los schemas con `drizzle-zod` (`createInsertSchema`/`createSelectSchema`) en vez de `z.object` manual
- Con RHF usa `zodResolver` para sincronizar errores
- Con tRPC los schemas Zod son la fuente de tipos de extremo a extremo