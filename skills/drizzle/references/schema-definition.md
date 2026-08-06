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

## Buenas prácticas

- Una tabla por archivo o agrupar en `src/db/schema.ts`
- Exportar `* as schema` para pasarlo a `drizzle()` y `defineRelations`
- Nombres de columnas en snake_case con alias opcional: `timestamp('created_at')`
- Usar `$type<T>()` para columnas JSON en lugar de tipos sueltos
- Siempre definir PK y los índices que usen tus queries más frecuentes