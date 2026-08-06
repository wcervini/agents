# Schema Definition con Drizzle

Definición de tablas, columnas, índices y constraints, e inferencia de tipos.

## Tablas por dialecto

```typescript
// PostgreSQL
import { pgTable, serial, integer, text, boolean, timestamp, varchar, jsonb } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  verified: boolean('verified').notNull().default(false),
  metadata: jsonb('metadata').$type<string[]>(),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
});
```

```typescript
// SQLite
import { sqliteTable, integer, text } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
});
```

```typescript
// MySQL
import { mysqlTable, int, varchar, boolean, timestamp } from 'drizzle-orm/mysql-core';

export const users = mysqlTable('users', {
  id: int('id').autoincrement().primaryKey(),
  name: varchar('name', { length: 255 }).notNull(),
  verified: boolean('verified').default(false),
  createdAt: timestamp('created_at').defaultNow(),
});
```

## Columnas comunes

| Columna | Uso |
|---------|-----|
| `serial()` | Autoincrement entero (Postgres) |
| `integer()` / `int()` | Entero |
| `text()` | Texto largo |
| `varchar('n', { size })` / `varchar('n', { length })` | Texto con límite |
| `boolean()` | Booleano |
| `timestamp('c', { withTimezone })` | Fecha/hora |
| `date()` | Solo fecha |
| `jsonb().$type<T>()` | JSON tipado (Postgres) |
| `uuid()` | UUID (Postgres) |

Modificadores: `.notNull()`, `.default(v)`, `.unique()`, `.primaryKey()`, `.$type<T>()`, `.references(() => tabla.columna)`.

## Índices y constraints

```typescript
import { integer, text, index, uniqueIndex, sqliteTable } from 'drizzle-orm/sqlite-core';

export const user = sqliteTable('user', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  name: text('name'),
  email: text('email'),
}, (table) => [
  index('name_idx').on(table.name),
  uniqueIndex('email_idx').on(table.email),
]);
```

```sql
CREATE TABLE `user` ( ... );
CREATE INDEX `name_idx` ON `user` (`name`);
CREATE UNIQUE INDEX `email_idx` ON `user` (`email`);
```

### Primary key compuesta y foreign keys

```typescript
// PK compuesta
export const membership = pgTable('membership', {
  userId: integer('user_id').notNull().references(() => users.id),
  teamId: integer('team_id').notNull().references(() => teams.id),
}, (t) => [
  primaryKey({ columns: [t.userId, t.teamId] }),
  index('membership_user_idx').on(t.userId),
]);
```

## Inferencia de tipos

```typescript
import { integer, text, sqliteTable } from 'drizzle-orm/sqlite-core';
import { type InferSelectModel, type InferInsertModel } from 'drizzle-orm';

const users = sqliteTable('users', {
  id: integer().primaryKey({ autoIncrement: true }),
  name: text().notNull(),
});

// Notación $infer (preferida)
type SelectUser = typeof users.$inferSelect;   // { id: number; name: string }
type InsertUser = typeof users.$inferInsert;   // { id?: number; name: string }

// Notación con tipos genéricos (equivalente)
type SelectUserAlt = InferSelectModel<typeof users>;
type InsertUserAlt = InferInsertModel<typeof users>;
```

- `$inferSelect` → fila leída de la base (todas las columnas como definidas)
- `$inferInsert` → fila a insertar (PK y columnas con default son opcionales)

## SQLite: fechas y booleanos (integer + mode)

SQLite **no tiene tipos nativos `BOOLEAN` ni `DATETIME`**. En stacks con Drizzle, SQLite se maneja vía el ORM (no con SQL puro suelto), y Drizzle abstrae esos tipos con `integer` + `mode`:

| Concepto | Columna Drizzle | SQLite subyacente | JS |
|----------|----------------|-------------------|----|
| Booleano | `integer('col', { mode: 'boolean' })` | `INTEGER` 0/1 | `true`/`false` |
| Fecha | `integer('col', { mode: 'timestamp_ms' })` | `INTEGER` Unix ms | `Date` |
| Fecha | `integer('col', { mode: 'timestamp' })` | `INTEGER` Unix seg | `Date` |

```typescript
import { sqliteTable, integer, text } from 'drizzle-orm/sqlite-core';

export const user = sqliteTable('user', {
  id: text('id').primaryKey(),
  email: text('email').notNull().unique(),
  emailVerified: integer('email_verified', { mode: 'boolean' }).default(false).notNull(),
  createdAt: integer('created_at', { mode: 'timestamp_ms' }).notNull(),
});
```

> ❌ NO uses `TEXT` ISO (string ISO) ni `DATETIME` para fechas/booleanos en el schema de Drizzle.

## Derivar schemas zod (drizzle-zod)

Drizzle es la fuente de verdad del esquema; los schemas de validación zod se **derivan** con `drizzle-zod`:

```js
import { createInsertSchema, createSelectSchema } from 'drizzle-zod';
import { z } from 'zod';
import { productos } from '../db/schema.js';

export const insertarProductoSchema = createInsertSchema(productos, {
  precio: z.number().int().positive('El precio debe ser mayor a 0'),
  stock: z.number().int().min(0, 'El stock no puede ser negativo'),
});

export const seleccionarProductoSchema = createSelectSchema(productos);
```

- `createInsertSchema(tabla, refinamientos?)` → schema para INSERT (el 2º argumento sobreescribe solo campos concretos)
- `createSelectSchema(tabla)` → schema para SELECT, sin sobreescribir
- Ver skill `zod` → `references/integrations.md` para el detalle de la integración

## Buenas prácticas

- Una tabla por archivo o agrupar en `src/db/schema.ts`
- Exportar `* as schema` para pasarlo a `drizzle()` y `defineRelations`
- Nombres de columnas en snake_case con alias opcional: `timestamp('created_at')`
- Usar `$type<T>()` para columnas JSON en lugar de tipos sueltos
- Siempre definir PK y los índices que usen tus queries más frecuentes