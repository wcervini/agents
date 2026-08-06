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
- Inferir tipos con `$inferSelect`/`$inferInsert` o `InferSelectModel`/`InferInsertModel`
- Definir PK y los índices que usen las queries más frecuentes
- Usar `$type<T>()` para columnas JSON
- Cifrar secrets en `dbCredentials` con `process.env.*`

## MUST NOT DO

- Usar `any` — Drizzle es type-safe por diseño
- Hardcodear credenciales en `drizzle.config.ts`
- Confundir `$inferSelect` (lectura) con `$inferInsert` (escritura)
- Olvidar `.notNull()`/`.default()` cuando corresponda

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