# Integraciones de Drizzle

Cómo integrar Drizzle ORM con frameworks, servicios edge y Better Auth.

## Tabla de integraciones

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

## Better Auth (drizzleAdapter)

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

Genera el schema de auth:

```bash
npx @better-auth/cli generate --output src/db/auth-schema.ts
npx drizzle-kit push        # dev
# o en producción:
npx drizzle-kit generate && npx drizzle-kit migrate
```

> Ver skills `better-auth-best-practices` y `create-auth` para la configuración completa de Better Auth.

## Turso / LibSQL (edge)

```typescript
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client';

const client = createClient({ url: process.env.TURSO_DATABASE_URL!, authToken: process.env.TURSO_AUTH_TOKEN! });
const db = drizzle(client);
```

## Neon (serverless)

```typescript
import { drizzle } from 'drizzle-orm/neon-serverless';
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);
const db = drizzle({ client: sql });
```

## PlanetScale

```typescript
import { drizzle } from 'drizzle-orm/planetscale-serverless';

const db = drizzle({
  connection: {
    host: process.env.DATABASE_HOST!,
    username: process.env.DATABASE_USERNAME!,
    password: process.env.DATABASE_PASSWORD!,
  },
});
```

## tRPC

```typescript
import { initTRPC } from '@trpc/server';
import { z } from 'zod';
import { db } from './db';
import { users } from './schema';
import { eq } from 'drizzle-orm';

const t = initTRPC.create();

export const appRouter = t.router({
  getUser: t.procedure
    .input(z.object({ id: z.number() }))
    .query(async ({ input }) => {
      const [user] = await db.select().from(users).where(eq(users.id, input.id));
      return user;
    }),
});
```

## Hono

```typescript
import { Hono } from 'hono';
import { db } from './db';
import { users } from './db/schema';

const app = new Hono();

app.get('/users', async (c) => {
  const all = await db.select().from(users);
  return c.json(all);
});
```

## Astro

```typescript
// src/pages/api/users.ts
import type { APIRoute } from 'astro';
import { db } from '../../db';
import { users } from '../../db/schema';

export const GET: APIRoute = async () => {
  const all = await db.select().from(users);
  return new Response(JSON.stringify(all), { headers: { 'Content-Type': 'application/json' } });
};
```

## Rendering en edge/serverless

- Drizzle es headless y funciona en edge/serverless (Neon, Turso, PlanetScale)
- Usa drivers serverless (`@neondatabase/serverless`, `@libsql/client`, `@planetscale/database`)
- Evita conexiones persistentes en serverless; deja que el driver gestione el pool

## Buenas prácticas

- Cifrar secrets en env vars, nunca en código
- Elegir el driver según el entorno (local vs edge)
- Con Better Auth, el `provider` del adapter debe coincidir con el driver
- Preparar statements para queries hot-path en serverless