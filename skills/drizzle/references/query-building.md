# Query Building y CRUD con Drizzle

Queries tipadas, operadores, joins, CRUD, agregaciones y prepared statements.

## Instancia db

```typescript
// PostgreSQL (node-postgres)
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle({ client: pool, schema });
```

```typescript
// SQLite (better-sqlite3)
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';

const sqlite = new Database('sqlite.db');
const db = drizzle({ client: sqlite });
```

## SELECT

```typescript
import { eq, and, or, inArray, like, between, isNotNull, count } from 'drizzle-orm';

// Todos
const all = await db.select().from(users);

// Con WHERE
const verified = await db.select().from(users).where(eq(users.verified, true));

// Operadores combinados
const result = await db
  .select()
  .from(users)
  .where(
    and(
      eq(users.verified, true),
      or(eq(users.role, 'admin'), eq(users.role, 'moderator')),
    ),
  );

// inArray / like / between
await db.select().from(users).where(inArray(users.id, [1, 2, 3]));
await db.select().from(users).where(like(users.name, '%ana%'));
await db.select().from(users).where(between(users.age, 18, 65));

// isNull / isNotNull
await db.select().from(users).where(isNull(users.deletedAt));

// Selección de columnas específicas
const names = await db.select({ id: users.id, name: users.name }).from(users);
```

## Joins

```typescript
await db
  .select()
  .from(posts)
  .leftJoin(comments, eq(posts.id, comments.post_id))
  .where(eq(posts.id, 10));
```

```sql
SELECT * FROM `posts`
LEFT JOIN `comments` ON `posts`.`id` = `comments`.`post_id`
WHERE `posts`.`id` = 10
```

Joins disponibles: `.innerJoin`, `.leftJoin`, `.rightJoin`, `.fullJoin`.

## CRUD

```typescript
// INSERT
const [user] = await db
  .insert(users)
  .values({ name: 'Ana', email: 'ana@x.com' })
  .returning(); // devuelve la fila insertada

// Upsert
await db
  .insert(users)
  .values({ email: 'ana@x.com', name: 'Ana' })
  .onConflictDoNothing();
// o
await db
  .insert(users)
  .values({ email: 'ana@x.com', name: 'Ana' })
  .onConflictDoUpdate({ target: users.email, set: { name: 'Ana' } });

// UPDATE
await db
  .update(users)
  .set({ age: 31 })
  .where(eq(users.email, 'ana@x.com'));

// DELETE
await db.delete(users).where(eq(users.email, 'ana@x.com'));
```

## Agregaciones

```typescript
import { count, sum, avg, min, max } from 'drizzle-orm';

const total = await db.select({ count: count() }).from(users);
const avgAge = await db.select({ avg: avg(users.age) }).from(users);
```

## Prepared statements

```typescript
import { sql } from 'drizzle-orm';

const stmt = db
  .select()
  .from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('getUserById');

await stmt.execute({ id: 1 });
```

Prepared statements mejoran mucho el rendimiento en queries hot-path (reutilizan el plan de ejecución).

## Buenas prácticas

- Evitar N+1: usa `with` en `db.query` (relational queries) o joins
- Preparar statements para queries repetidas
- Usar `and`/`or` para condiciones combinadas
- Usar `.returning()` cuando necesites la fila resultante
- Filtrar en la base (`.where`) en lugar de en memoria