# Relations y Transacciones con Drizzle

Relaciones v2 (`defineRelations`), relational queries (`db.query`) y transacciones.

> ⚠️ **Relational Queries v1 (`relations()`) fue REMOVIDO.** Usa `defineRelations()` (v2).

## defineRelations (v2)

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

const db = drizzle(client, { relations }); // ⚠️ pasar { relations }
```

## Relational queries — db.query

```typescript
const result = await db.query.posts.findMany({
  with: {
    author: true,
  },
});
// [{ id: 10, content: '...', author: { id: 1, name: 'Alex' } }]
```

### Relaciones 1:N y filtros predefinidos

```typescript
export const relations = defineRelations(schema, (r) => ({
  groups: {
    verifiedUsers: r.many.users({
      from: r.groups.id.through(r.usersToGroups.groupId),
      to: r.users.id.through(r.usersToGroups.userId),
      where: { verified: true },
    }),
  },
}));

const response = await db.query.groups.findMany({
  with: { verifiedUsers: true },
});
```

### limit/offset en `with` (nuevo en v2)

```typescript
await db.query.posts.findMany({
  limit: 5,
  offset: 2,
  with: {
    comments: { offset: 3, limit: 3 },
  },
});
```

### Operadores en relational queries

```typescript
await db.query.users.findFirst({
  where: (table, { inArray }) => inArray(table.id, [1, 2, 3]),
});
```

## Transacciones

```typescript
await db.transaction(async (tx) => {
  const [user] = await tx.insert(users).values({ name: 'Ana' }).returning();
  await tx.insert(posts).values({ content: 'Hola', ownerId: user.id });
  // si algo lanza, se hace rollback de todo
});
```

- Usa `tx` (no `db`) dentro de la transacción
- Si una query lanza, toda la transacción se revierte
- Para operaciones atómicas multi-paso (transferencias, creación con dependencias)

## Batch

Con drivers compatibles (libsql, neon) puedes agrupar queries:

```typescript
const [a, b] = await db.batch([
  db.select().from(users),
  db.select().from(posts),
]);
```

## Buenas prácticas

- Pasar `{ relations }` a `drizzle()` si usas `db.query`
- Usar `with` para eager loading y evitar N+1
- Usar transacciones para operaciones atómicas multi-paso
- Preferir `db.query...findMany({ with })` para datos anidados tipados