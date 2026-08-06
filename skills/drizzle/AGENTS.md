# Drizzle ORM Cheatsheet

**Version:** 1.0.0 | **Drizzle ORM 0.44+ / Kit** | **Updated:** 2026-08-06 | **Author**: delineas

---

## Quick Reference Card

| Tema | Regla práctica |
|------|---------------|
| **Instalación** | `npm i drizzle-orm` + `npm i -D drizzle-kit`; drivers: `pg`, `postgres`, `@neondatabase/serverless`, `mysql2`, `better-sqlite3`, `@libsql/client`, `@planetscale/database` |
| **Config** | `drizzle.config.ts` con `defineConfig({ schema, out, dialect, dbCredentials })` |
| **Dialectos** | `postgresql` · `mysql` · `sqlite` · `turso` · `singlestore` · `cockroachdb` |
| **CLI** | `generate` (SQL versionado), `migrate` (aplica), `push` (sync dev), `studio` (UI), `check`, `pull`, `up` |
| **Tablas** | `pgTable('users', {...})`, `sqliteTable`, `mysqlTable` |
| **Columnas** | `serial()`, `integer()`, `text()`, `boolean()`, `timestamp('c', { withTimezone: true })`, `varchar('n', { size: 256 })`, `jsonb().$type<T>()` |
| **Índices/constraints** | `index('idx').on(col)`, `uniqueIndex('u').on(col)`, `.primaryKey()`, `.unique()`, `.notNull()`, `.default(v)`, `foreignKey()` |
| **Tipos** | `typeof users.$inferSelect` / `$inferInsert` · `InferSelectModel<typeof users>` / `InferInsertModel<typeof users>` |
| **CRUD** | `db.insert(t).values(v).returning()` · `db.update(t).set(v).where(cond)` · `db.delete(t).where(cond)` |
| **Queries** | `db.select().from(t).where(eq(col, v))`; operadores: `eq`, `and`, `or`, `inArray`, `like`, `between`, `isNotNull`, `count` |
| **Joins** | `.innerJoin(t2, on)`, `.leftJoin(t2, on)`, `.rightJoin`, `.fullJoin` |
| **Relaciones** | ⚠️ v1 `relations()` REMOVIDO → `defineRelations(schema, r => ({...}))` con `r.one`/`r.many`; consultar con `db.query.t.findMany({ with })` |
| **Transacciones** | `db.transaction(async (tx) => { ... })` — usa `tx` para queries atómicas |
| **Prepared** | `const stmt = db.select().from(t).where(eq(...)).prepare('name'); await stmt.execute(...)` |
| **Better Auth** | `import { drizzleAdapter } from 'better-auth/adapters/drizzle'` + `provider: 'pg' \| 'mysql' \| 'sqlite'` |
| **drizzle-zod** | `npm i drizzle-zod` → `createInsertSchema(tabla, {campo: refinamiento})` / `createSelectSchema(tabla)` |
| **SQLite modes** | Booleano: `integer('c', { mode: 'boolean' })` · Fecha: `integer('c', { mode: 'timestamp_ms' })` (ms) / `{ mode: 'timestamp' }` (seg) |
| **Ref. raíz** | https://orm.drizzle.team/docs/ |

---

## Decision Trees

### Driver por dialecto
```
SQLite local → better-sqlite3 → drizzle-orm/better-sqlite3
SQLite edge  → @libsql/client → drizzle-orm/libsql (Turso)
PostgreSQL   → pg            → drizzle-orm/node-postgres
PostgreSQL   → postgres.js   → drizzle-orm/postgres-js
PostgreSQL edge → @neondatabase/serverless → drizzle-orm/neon-serverless
MySQL        → mysql2        → drizzle-orm/mysql2
PlanetScale  → @planetscale/database → drizzle-orm/planetscale-serverless
```

### generate vs push
```
¿Producción o equipo (migraciones versionadas)?
├── Sí → drizzle-kit generate && drizzle-kit migrate
└── Dev/prototyping → drizzle-kit push
```

### Relaciones: joins vs defineRelations
```
¿Necesitas datos anidados tipados (author dentro de posts)?
├── Sí → defineRelations + db.query.posts.findMany({ with: { author: true } })
└── No → SQL-style: db.select().from(posts).leftJoin(comments, eq(...))
```

---

## Critical "NEVER Do" List

- **NEVER** usar `relations()` v1 — **REMOVIDO**; usa `defineRelations()` (v2)
- **NEVER** hacer N+1 — usa `with` en `db.query` o joins
- **NEVER** hardcodear credenciales en `drizzle.config.ts` — usa `process.env.DATABASE_URL`
- **NEVER** usar `any` — Drizzle es type-safe por diseño
- **NEVER** usar `drizzle-kit push` como estrategia de producción — usa migraciones
- **NEVER** olvidar pasar `{ relations }` a `drizzle()` si usas `db.query`
- **NEVER** confundir `$inferSelect` (lectura) con `$inferInsert` (escritura)
- **NEVER** ignorar la columna `id` al hacer `.where()` — usa `eq(users.id, id)`
- **NEVER** definir el schema con `CREATE TABLE` suelto si el proyecto usa Drizzle — el schema vive en `sqliteTable`/`pgTable`
- **NEVER** usar `TEXT` ISO o `DATETIME` para fechas/booleanos en SQLite — usa `integer` con `{ mode }`

---

## Drizzle v2 Notes (Post-Training / Verificado con Context7)

### Relational Queries v2 — defineRelations()

```typescript
import { drizzle } from 'drizzle-orm/…';
import { defineRelations } from 'drizzle-orm';
import * as p from 'drizzle-orm/pg-core';

export const users = p.pgTable('users', {
  id: p.integer().primaryKey(),
  name: p.text().notNull(),
});

export const posts = p.pgTable('posts', {
  id: p.integer().primaryKey(),
  content: p.text().notNull(),
  ownerId: p.integer('owner_id'),
});

const relations = defineRelations({ users, posts }, (r) => ({
  posts: {
    author: r.one.users({
      from: r.posts.ownerId,
      to: r.users.id,
    }),
  },
}));

const db = drizzle(client, { relations });

const result = await db.query.posts.findMany({
  with: { author: true },
});
```

- `r.one.users({ from: r.posts.ownerId, to: r.users.id })` → relación 1:1/N:1
- `r.many.posts()` → relación 1:N
- Filtros predefinidos: `r.many.users({ where: { verified: true } })` y query con `verifiedUsers: true`
- `limit`/`offset` SÍ funcionan en `with` (nuevo en v2)
- **Nota migración:** el operador `inArray` en queries relacionales ahora se importa del segundo argumento: `where: (table, { inArray }) => inArray(table.id, [...])`

### Migraciones runtime

```typescript
// Aplicar migraciones desde código (Node/Postgres)
import { migrate } from 'drizzle-orm/node-postgres/migrator';
import { db } from './db';
import { pool } from './db/pool';

await migrate(db, { migrationsFolder: './drizzle' });
await pool.end();
```

### Preparación de statements

```typescript
const stmt = db
  .select()
  .from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('getUserById');

await stmt.execute({ id: 1 });
```

### drizzleAdapter (Better Auth)

```typescript
import { betterAuth } from 'better-auth';
import { drizzleAdapter } from 'better-auth/adapters/drizzle';
import { db } from './db';

export const auth = betterAuth({
  database: drizzleAdapter(db, {
    provider: 'pg', // 'pg' | 'mysql' | 'sqlite' — debe coincidir con el driver
  }),
  emailAndPassword: { enabled: true },
});
```

> Genera el schema de auth con: `npx @better-auth/cli generate --output src/db/auth-schema.ts`, luego `npx drizzle-kit push` (dev) o `generate && migrate` (prod). Ver skills `better-auth-best-practices` y `create-auth`.

---

## Integraciones (mínimas)

| Integración | Paquete/API | Ejemplo mínimo |
|-------------|-------------|----------------|
| Better Auth | `better-auth/adapters/drizzle` | `database: drizzleAdapter(db, { provider: 'pg' })` |
| tRPC | `drizzle-orm` directo | `query: async () => db.select().from(users)` |
| Hono | `drizzle-orm` directo | handler con `db` importada |
| Astro | `drizzle-orm/*` drivers | `src/pages/api/*.ts` con `db` |
| Next.js | `drizzle-orm/neon-serverless` | route handlers con `db` |
| Turso/LibSQL | `drizzle-orm/libsql` + `@libsql/client` | `drizzle(createClient({ url }))` |
| Neon | `drizzle-orm/neon-serverless` + `@neondatabase/serverless` | `drizzle(neon(process.env.DATABASE_URL!))` |
| PlanetScale | `drizzle-orm/planetscale-serverless` | `drizzle({ connection: {...} })` |

---

## Key Patterns (Brief)

- **Upsert:** `db.insert(t).values(v).onConflictDoNothing()` / `.onConflictDoUpdate({ target: t.email, set: {...} })`
- **Retorno:** `.returning()` devuelve filas insertadas/actualizadas (Postgres/MySQL; SQLite requiere driver)
- **Agregaciones:** `db.select({ count: count() }).from(t)` — `count()` desde `drizzle-orm`
- **Like:** `like(t.name, '%ana%')`; **Between:** `between(t.age, 18, 65)`
- **AND/OR:** `and(eq(a, 1), or(eq(b, 2), eq(b, 3)))`
- **Batch:** múltiples `db.batch([...])` con drivers compatibles (libsql, neon)
- **`$type<T>()`** para columnas especiales: `jsonb().$type<string[]>()`
- **Schema en Drizzle, validación derivada:** SQLite se maneja vía Drizzle ORM (no SQL puro suelto) y los schemas zod se derivan con `createInsertSchema`/`createSelectSchema` (drizzle-zod)

## References

Docs a detalle con ejemplos completos en `references/`:

`schema-definition.md` `migrations.md` `query-building.md` `relations.md` `integrations.md`
