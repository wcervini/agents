---
description: Reglas de queries, CRUD y performance con Drizzle ORM
globs:
  - "**/*.ts"
  - "**/*.tsx"
---

# Drizzle Query Rules

## MUST DO

- Usar `db.select().from().where()` con operadores (`eq`, `and`, `or`, `inArray`, `like`, `between`)
- Usar `db.transaction(async (tx) => {})` para operaciones atómicas multi-paso
- Usar `with` en `db.query` o joins para evitar N+1
- Usar prepared statements (`.prepare()`) para queries hot-path
- Usar `.returning()` cuando necesites la fila resultante

## MUST NOT DO

- Hacer N+1 — usa `with` en `db.query` o joins
- Usar `relations()` v1 (removido) — usa `defineRelations` (v2)
- Olvidar pasar `{ relations }` a `drizzle()` si usas `db.query`
- Filtrar en memoria en lugar de en la base (`.where`)
- Usar `db` en lugar de `tx` dentro de una transacción

## Patrones

### CRUD

```typescript
const [user] = await db.insert(users).values({ name: 'Ana' }).returning();
await db.update(users).set({ age: 31 }).where(eq(users.email, 'ana@x.com'));
await db.delete(users).where(eq(users.email, 'ana@x.com'));
```

### Transacción

```typescript
await db.transaction(async (tx) => {
  const [user] = await tx.insert(users).values({ name: 'Ana' }).returning();
  await tx.insert(posts).values({ content: 'Hola', ownerId: user.id });
});
```

### Prepared statement

```typescript
const stmt = db
  .select()
  .from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('getUserById');

await stmt.execute({ id: 1 });
```