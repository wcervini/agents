# Migraciones con Drizzle Kit

Configuración de `drizzle.config.ts` y comandos CLI de Drizzle Kit.

## drizzle.config.ts

```typescript
import { defineConfig } from 'drizzle-kit';

export default defineConfig({
  schema: './src/db/schema.ts',   // ruta al schema
  out: './drizzle',               // carpeta de migraciones
  dialect: 'postgresql',          // 'postgresql' | 'mysql' | 'sqlite' | 'turso' | 'singlestore' | 'cockroachdb'
  dbCredentials: {
    url: process.env.DATABASE_URL!,   // nunca hardcodear secrets
  },
});
```

Para SQLite local:

```typescript
export default defineConfig({
  schema: './src/db/schema.ts',
  out: './drizzle',
  dialect: 'sqlite',
  dbCredentials: {
    url: './sqlite.db',
  },
});
```

## Comandos CLI

```bash
# Genera archivos SQL versionados a partir del schema (en out/)
drizzle-kit generate

# Aplica las migraciones pendientes a la base
drizzle-kit migrate

# Sincroniza schema ↔ DB directamente (dev/prototyping, sin archivos)
drizzle-kit push

# Extrae schema desde la base (reverse engineering)
drizzle-kit pull

# Verifica la configuración
drizzle-kit check

# Actualiza el kit
drizzle-kit up

# UI web para explorar/editar datos
drizzle-kit studio

# Exporta datos
drizzle-kit export
```

También puedes pasar opciones por CLI:

```bash
npx drizzle-kit generate --dialect=postgresql --schema=./src/schema.ts
```

## Flujo recomendado

### Desarrollo

```bash
drizzle-kit generate   # crea SQL en out/
drizzle-kit migrate    # aplica a la DB local
# o directamente:
drizzle-kit push       # sync rápido sin archivos
```

### Producción

```bash
drizzle-kit generate   # versiona los cambios
drizzle-kit migrate    # aplica en el entorno de producción
```

## Migraciones runtime (desde código)

```typescript
import { migrate } from 'drizzle-orm/node-postgres/migrator';
import { db } from './db';
import { pool } from './db/pool';

await migrate(db, { migrationsFolder: './drizzle' });
await pool.end();
```

Otros migrators: `drizzle-orm/better-sqlite3/migrator`, `drizzle-orm/libsql/migrator`, `drizzle-orm/mysql2/migrator`, `drizzle-orm/neon-serverless/migrator`.

## Buenas prácticas

- Cifrar secrets: `dbCredentials.url` desde `process.env`, nunca hardcodeado
- Versionar la carpeta `out/` (migraciones) en git
- En CI/CD, ejecutar `drizzle-kit migrate` como paso de deploy
- `push` es cómodo en dev pero no versionable — usa migraciones para producción
- Revisar el SQL generado antes de aplicarlo en producción