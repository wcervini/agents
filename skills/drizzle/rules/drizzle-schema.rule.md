---
description: Reglas de definición de schema y tipos con Drizzle ORM
globs:
  - "**/*.ts"
  - "drizzle.config.ts"
  - "**/schema.ts"
---

# Drizzle Schema Rules

## MUST DO

- Usar `defineConfig` en `drizzle.config.ts` con `dialect`, `schema`, `out`, `dbCredentials`
- Definir el esquema **en Drizzle** (`sqliteTable`/`pgTable`/`mysqlTable`) — no dejar SQL suelto (`CREATE TABLE`) cuando el proyecto usa Drizzle
- Inferir tipos con `$inferSelect`/`$inferInsert` o `InferSelectModel`/`InferInsertModel`
- En SQLite, fechas como `integer('c', { mode: 'timestamp_ms' })` y booleanos como `integer('c', { mode: 'boolean' })` — NO `TEXT` ISO ni `DATETIME`
- Derivar schemas zod con `createInsertSchema`/`createSelectSchema` (drizzle-zod) en vez de `z.object` a mano
- Definir PK y los índices que usen las queries más frecuentes
- Usar `$type<T>()` para columnas JSON
- Cifrar secrets en `dbCredentials` con `process.env.*`

## MUST NOT DO

- Usar `any` — Drizzle es type-safe por diseño
- Hardcodear credenciales en `drizzle.config.ts`
- Confundir `$inferSelect` (lectura) con `$inferInsert` (escritura)
- Olvidar `.notNull()`/`.default()` cuando corresponda
- Escribir `CREATE TABLE` suelto si el esquema ya se define en Drizzle
- Usar `TEXT`/`DATETIME` para fechas o booleanos en SQLite (usa `integer` + `mode`)

## Patrones

### Tabla PostgreSQL

```typescript
import { pgTable, serial, text, boolean, timestamp, jsonb } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  verified: boolean('verified').notNull().default(false),
  metadata: jsonb('metadata').$type<string[]>(),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
});
```

### Inferencia de tipos

```typescript
type SelectUser = typeof users.$inferSelect;
type InsertUser = typeof users.$inferInsert;
```

### SQLite: fechas y booleanos (integer + mode)

SQLite no tiene `BOOLEAN` ni `DATETIME` nativos; Drizzle los abstrae con `integer` + `mode`:

```typescript
import { sqliteTable, integer, text } from 'drizzle-orm/sqlite-core';

export const usuario = sqliteTable('usuario', {
  id: text('id').primaryKey(),
  emailVerified: integer('email_verified', { mode: 'boolean' }).default(false).notNull(),
  createdAt: integer('created_at', { mode: 'timestamp_ms' }).notNull(),
});
```

### Derivar schemas zod con drizzle-zod

El schema vive en Drizzle; los schemas de validación se derivan (no `z.object` a mano):

```js
import { createInsertSchema, createSelectSchema } from 'drizzle-zod';
import { z } from 'zod';
import { productos } from '../db/schema.js';

export const insertarProductoSchema = createInsertSchema(productos, {
  precio: z.number().int().positive('El precio debe ser mayor a 0'),
});

export const seleccionarProductoSchema = createSelectSchema(productos);
```