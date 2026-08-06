# Parsing con Zod

Guía de parsing de datos y composición de esquemas con Zod.

## Instalación

Zod es **solo una librería** (no tiene CLI ni binario):

```bash
npm i zod        # npm
pnpm add zod     # pnpm
bun add zod      # bun
```

## Primitivas

```typescript
import { z } from 'zod';

z.string();                    // string
z.string().min(2).max(50);     // con longitud
z.string().email();            // email válido
z.string().url();              // URL válida
z.string().uuid();             // UUID
z.string().regex(/^[a-z]+$/);  // regex
z.number();                    // number
z.number().int();              // entero
z.number().positive();         // > 0
z.number().min(0).max(10);     // rango
z.boolean();                   // boolean
z.date();                      // Date (instancia real)
z.literal('admin');            // valor literal exacto
z.enum(['admin', 'user']);     // enum de strings
z.nativeEnum(Color);           // enum TS nativo
z.bigint();                    // bigint
z.symbol();                    // symbol
z.undefined();                 // undefined
z.null();                      // null
z.any();                       // cualquier cosa (evitar)
z.unknown();                   // desconocido (preferir)
z.never();                     // nunca (para exhaustividad)
```

## Composición

```typescript
// Objeto
const User = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  role: z.enum(['admin', 'user']).default('user'),
});

// Array
const Tags = z.array(z.string()).min(1).max(10);

// Unión
const Id = z.union([z.string(), z.number()]);

// Unión discriminada (campo discriminador)
const Event = z.discriminatedUnion('type', [
  z.object({ type: z.literal('click'), x: z.number() }),
  z.object({ type: z.literal('scroll'), y: z.number() }),
]);

// Tupla
const Point = z.tuple([z.number(), z.number()]);

// Record (mapa clave→valor)
const Scores = z.record(z.string(), z.number());

// Intersección
const Base = z.object({ id: z.number() });
const Extra = z.object({ name: z.string() });
const Combined = z.intersection(Base, Extra);

// Nullable / Optional
z.string().nullable();   // string | null
z.string().optional();   // string | undefined
z.string().nullish();    // string | null | undefined
```

## Métodos de parsing

```typescript
// Lanza ZodError si falla
const data = schema.parse(input);

// Devuelve resultado (no lanza)
const result = schema.safeParse(input);
if (result.success) {
  result.data;   // tipo inferido
} else {
  result.error;  // ZodError
}

// Versiones async (para validaciones/transforms async)
const data = await schema.parseAsync(input);
const result = await schema.safeParseAsync(input);
```

### Top-level parse (Zod Core, para autores de librerías)

```typescript
import { z as z4 } from 'zod/v4/core';

function parseData<T extends z4.$ZodType>(data: unknown, schema: T): z4.output<T> {
  return z4.parse(schema, data);
}
```

## Refinamiento y transformación

```typescript
// .refine — validación booleana simple
const Password = z.string().refine((s) => s.length >= 8, 'Mínimo 8 caracteres');

// .superRefine — múltiples issues con control fino
const Strings = z.array(z.string()).superRefine((val, ctx) => {
  if (val.length > 3) {
    ctx.addIssue({
      code: z.ZodIssueCode.too_big,
      maximum: 3,
      type: 'array',
      inclusive: true,
      message: 'Demasiados items',
    });
  }
  if (val.length !== new Set(val).size) {
    ctx.addIssue({ code: z.ZodIssueCode.custom, message: 'No se permiten duplicados' });
  }
});

// .transform — transforma el valor y el tipo de salida
const Slug = z.string().transform((s) => s.toLowerCase().replace(/\s+/g, '-'));

// z.pipe — encadena validaciones/transforms
const Num = z.pipe(z.string(), z.coerce.number());

// .catch — valor de respaldo ante error
const Safe = z.string().catch('default');

// .default — valor por defecto si el input es undefined
const Role = z.enum(['admin', 'user']).default('user');
```

## Buenas prácticas

- Validar en el **borde de la API** (bodies, queries, params, env vars, config)
- Usar `safeParse` en flujos controlados; `parse` solo para invariantes
- Mensajes de error claros y en el idioma del usuario final
- Nunca confiar en datos sin validar