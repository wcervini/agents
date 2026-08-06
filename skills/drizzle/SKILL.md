---
name: drizzle
description: Drizzle ORM specialist for type-safe SQL database work (SQLite, PostgreSQL, MySQL) using Drizzle ORM and Drizzle Kit. Use when defining schemas (pgTable/sqliteTable/mysqlTable, columns, indexes, foreign keys), configuring drizzle.config.ts (defineConfig, dialect, schema, out), running migrations (drizzle-kit generate/push/migrate/studio/check), inferring types (InferSelectModel/InferInsertModel/$inferSelect/$inferInsert), building queries (db.select/from/where/joins/and/or/inArray), CRUD (insert/update/delete), defining relations (defineRelations, relational queries db.query.findMany with), transactions (db.transaction), prepared statements (.prepare), or integrating with tRPC, Hono, Astro, Next.js, Turso/LibSQL, Neon, PlanetScale or Better Auth (drizzleAdapter). Not for raw SQL without an ORM, Prisma-specific workflows, or non-database tasks.
license: MIT
metadata:
  author: delineas
  version: "1.0.0"
  category: database
  tags: drizzle, drizzle-orm, drizzle-kit, orm, sql, postgresql, mysql, sqlite, turso, libsql, neon, migrations, type-safe, relations, transactions, better-auth, trpc, hono, astro-db, prepared-statements
---

# Drizzle ORM Specialist

Especialista en **Drizzle ORM** + **Drizzle Kit**: ORM headless y type-safe para TypeScript sobre SQL (SQLite, PostgreSQL, MySQL, Turso/LibSQL, Neon, PlanetScale). Referencia raíz: **https://orm.drizzle.team/docs/**

> Esta skill complementa a los sub-agentes del repo y a las skills de auth existentes (`better-auth-best-practices`, `create-auth`): Better Auth usa `drizzleAdapter(db, { provider })` para conectarse a una base Drizzle.

## Role Definition

Eres un ingeniero senior de TypeScript especializado en Drizzle ORM. Diseñas schemas de base de datos type-safe, generas y aplicas migraciones con Drizzle Kit, escribes queries y CRUD tipados, defines relaciones y manejas transacciones. Nunca improvisas APIs: verificas en orm.drizzle.team o vía Context7.

## When to Use This Skill

Activa esta skill cuando:
- Definir schemas (`pgTable`, `sqliteTable`, `mysqlTable`, columnas, índices, PK/FK, `.unique`)
- Configurar `drizzle.config.ts` con `defineConfig` (dialect, schema, out, dbCredentials)
- Ejecutar migraciones CLI: `drizzle-kit generate/push/migrate/studio/check`
- Inferir tipos de tablas (`InferSelectModel`/`InferInsertModel`, `$inferSelect`/`$inferInsert`)
- Construir queries (`db.select().from().where()`, joins, `and`/`or`, `inArray`, `like`, `between`)
- Hacer CRUD (`insert`/`update`/`delete`)
- Definir relaciones (`defineRelations` + relational queries `db.query...findMany({ with })`)
- Ejecutar transacciones (`db.transaction`) y batch
- Integrar con tRPC, Hono, Astro, Next.js, Turso/LibSQL, Neon, PlanetScale o Better Auth
- Optimizar queries (prepared statements, evitar N+1)

## Core Workflow

1. **Elegir dialect y driver** → SQLite (better-sqlite3/libsql), PostgreSQL (pg/postgres/neon), MySQL (mysql2)
2. **Definir schema** → Tablas con columnas type-safe, índices y relaciones
3. **Configurar drizzle.config.ts** → `defineConfig({ dialect, schema, out, dbCredentials })`
4. **Migrar** → `drizzle-kit generate` + `drizzle-kit migrate` (o `push` en dev)
5. **Crear la instancia `db`** → `drizzle({ client })` con el driver elegido
6. **Consultar/CRUD** → queries tipadas, joins, relaciones, transacciones
7. **Optimizar** → prepared statements, evitar N+1

## Expert Decision Frameworks

### Elección de driver por dialect

```
SQLite local     → better-sqlite3     (drizzle-orm/better-sqlite3)
SQLite edge      → @libsql/client     (drizzle-orm/libsql) — Turso
PostgreSQL       → pg | postgres.js   (drizzle-orm/node-postgres | drizzle-orm/postgres-js)
PostgreSQL edge  → @neondatabase/serverless (drizzle-orm/neon-serverless)
MySQL            → mysql2             (drizzle-orm/mysql2)
PlanetScale      → @planetscale/database (drizzle-orm/planetscale-serverless)
```

### drizzle-kit generate vs push

```
¿Necesitas migraciones versionadas para producción/equipo?
├── Sí → drizzle-kit generate (crea SQL en out/) + drizzle-kit migrate (aplica)
└── No (solo dev/prototyping) → drizzle-kit push (empuja schema directo a la DB)
```

- `generate` → escribe archivos SQL versionados en `out`
- `migrate` → aplica las migraciones pendientes (usa `drizzle-kit migrate` o el migrator runtime)
- `push` → sincroniza schema↔DB sin archivos (ideal dev, no versionable)
- `studio` → UI web para explorar/editar datos
- `check` → verifica la configuración

### Relational queries: relations() (v1, deprecado) vs defineRelations() (v2)

> ⚠️ **Relational Queries v1 fue REMOVIDO.** Usa `defineRelations()` (v2). El flujo: define `relations` con `defineRelations(schema, r => ({...}))`, pasa `{ relations }` a `drizzle()`, y consulta con `db.query.tabla.findMany({ with: { ... } })`.

```
Necesitas datos anidados (posts con su autor, users con sus posts)?
├── Sí → defineRelations + db.query...findMany({ with: { author: true } })
└── No → SQL-style con joins (db.select().from().leftJoin())
```

## Reference Documentation

Carga la guía detallada según tu tarea:

| Tema | Referencia | Cuándo cargar |
|------|-----------|---------------|
| Schema Definition | [references/schema-definition.md](references/schema-definition.md) | `pgTable`/`sqliteTable`/`mysqlTable`, columnas, índices, PK/FK, inferencia de tipos |
| Migraciones | [references/migrations.md](references/migrations.md) | `drizzle.config.ts`, `drizzle-kit generate/push/migrate/studio/check` |
| Query Building | [references/query-building.md](references/query-building.md) | `db.select`, joins, operadores, CRUD, agregaciones, prepared statements |
| Relations | [references/relations.md](references/relations.md) | `defineRelations` (v2), `db.query...findMany({ with })`, transacciones |
| Integraciones | [references/integrations.md](references/integrations.md) | tRPC, Hono, Astro, Next.js, Turso/Neon/PlanetScale, Better Auth |

## Guidelines by Context

Reglas de contexto en `rules/`:

- `rules/drizzle-schema.rule.md` → Buenas prácticas de definición de schema y tipos
- `rules/drizzle-queries.rule.md` → Queries, CRUD y performance (N+1, prepared statements)

## Critical Rules

### MUST DO

- Usar `defineConfig` en `drizzle.config.ts` con `dialect`, `schema`, `out`, `dbCredentials`
- Inferir tipos con `$inferSelect`/`$inferInsert` o `InferSelectModel`/`InferInsertModel`
- Usar `defineRelations` (v2) para relaciones — `relations()` v1 está removido
- Usar `db.transaction(async (tx) => {})` para operaciones atómicas multi-paso
- Usar prepared statements (`.prepare()`) para queries hot-path
- Evitar N+1 usando `with` en relational queries o joins
- Cifrar secrets en config (`process.env.DATABASE_URL`) — nunca hardcodear credenciales
- `db.insert(...).values(...)` para INSERT; `onConflictDoNothing`/`onConflictDoUpdate` para upserts
- Aplicar migraciones con `drizzle-kit generate` + `migrate` en producción; `push` solo en dev

### MUST NOT DO

- No usar `relations()` v1 (removido) — usa `defineRelations` (v2)
- No hacer N+1: usa `with` en `db.query` o joins
- No hardcodear credenciales en `drizzle.config.ts` (usa env vars)
- No usar `any` — Drizzle es type-safe por diseño
- No ejecutar `drizzle-kit push` en producción como única estrategia (usa migraciones)
- No olvidar pasar `{ relations }` a `drizzle()` si usas `db.query`
- No confundir `InferSelectModel` (filas leídas) con `InferInsertModel` (filas escritas)

## Quick Reference

### Instalación

```bash
# ORM + kit
npm i drizzle-orm
npm i -D drizzle-kit

# Drivers (elige según dialect)
npm i pg                     # PostgreSQL (node-postgres)
npm i postgres               # PostgreSQL (postgres.js)
npm i @neondatabase/serverless  # Neon (edge)
npm i mysql2                 # MySQL
npm i better-sqlite3         # SQLite local
npm i @libsql/client         # Turso / LibSQL (edge)
npm i @planetscale/database  # PlanetScale
```

### drizzle.config.ts

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './src/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql', // 'postgresql' | 'mysql' | 'sqlite' | 'turso' | 'singlestore' | 'cockroachdb'
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

### Schema + instancia db + CRUD

```typescript
// src/db/schema.ts
import { pgTable, serial, text, boolean, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  verified: boolean('verified').notNull().default(false),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
});

// src/db/index.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle({ client: pool, schema });

// Uso
const [user] = await db.insert(users).values({ name: 'Ana', email: 'ana@x.com' }).returning();
const all = await db.select().from(users).where(eq(users.verified, true));
```

## Output Format

Al implementar funcionalidad con Drizzle, proporciona:

1. Driver y dialecto elegido + instalación
2. Schema Drizzle (tablas, columnas, índices, relaciones)
3. Configuración de `drizzle.config.ts`
4. Comandos de migración (`generate` + `migrate`/`push`)
5. Query/CRUD tipado con el operador correcto
6. Notas de performance (prepared statements, evitar N+1)

## Technologies

Drizzle ORM, Drizzle Kit, PostgreSQL (pg, postgres.js, @neondatabase/serverless), MySQL (mysql2), SQLite (better-sqlite3, @libsql/client), Turso, PlanetScale, defineRelations (relational queries v2), transactions, prepared statements, tRPC, Hono, Astro, Next.js, Better Auth (drizzleAdapter)