# Inferencia de tipos con Zod

Cómo extraer tipos estáticos de los esquemas para mantener el ecosistema sincronizado.

## z.infer / z.output / z.input

```typescript
import { z } from 'zod';

const Player = z.object({
  username: z.string(),
  xp: z.number(),
});

// Tipo de SALIDA (el que usas en tu código)
type Player = z.infer<typeof Player>;
// { username: string; xp: number }

// Alias de z.infer
type PlayerOut = z.output<typeof Player>;

// Tipo de ENTRADA (antes de transform/coerce)
type PlayerIn = z.input<typeof Player>;
```

### Con transform, input ≠ output

```typescript
const Slug = z.string().transform((s) => s.toLowerCase().replace(/\s+/g, '-'));

type SlugIn = z.input<typeof Slug>;   // string (entrada)
type SlugOut = z.output<typeof Slug>; // string (salida transformada)
```

### Con coerce, el input es unknown

```typescript
const A = z.coerce.number();
type AInput = z.input<typeof A>;   // unknown (acepta "42" o 42)
type AOutput = z.output<typeof A>; // number

const B = z.coerce.number<number>();
type BInput = z.input<typeof B>;   // number
```

## z.coerce — coerción de tipos

```typescript
z.coerce.number();    // string | number → number
z.coerce.date();      // string | Date → Date  (muy usado en Astro frontmatter)
z.coerce.boolean();   // "true"/"false"/1/0 → boolean
z.coerce.bigint();
```

## z.lazy — recursión

TS no puede inferir tipos recursivos; declara el tipo manualmente y pásalo como hint:

```typescript
const baseCategorySchema = z.object({ name: z.string() });

type Category = z.infer<typeof baseCategorySchema> & {
  subcategories: Category[];
};

const categorySchema: z.ZodType<Category> = baseCategorySchema.extend({
  subcategories: z.lazy(() => categorySchema.array()),
});

categorySchema.parse({
  name: 'People',
  subcategories: [{ name: 'Politicians', subcategories: [] }],
}); // pasa
```

## z.record

```typescript
const Scores = z.record(z.string(), z.number());
// valida claves y valores; modo "strict" por defecto (clave no reconocida → error)

const Loose = z.record(z.string(), z.number(), { mode: 'loose' });
// pasa claves inválidas sin error

// Claves numéricas (acepta { "1": "a" } con fallback numérico)
const ById = z.record(z.number(), z.string());
```

## z.tuple

```typescript
const Point = z.tuple([z.number(), z.number()]);
type Point = z.infer<typeof Point>; // [number, number]
```

## z.intersection

```typescript
const Base = z.object({ id: z.number() });
const Extra = z.object({ name: z.string() });
const Combined = z.intersection(Base, Extra);
// { id: number; name: string }
```

## z.discriminatedUnion vs z.union

```typescript
// union — prueba cada esquema en orden
const Id = z.union([z.string(), z.number()]);

// discriminatedUnion — usa un campo discriminador literal
const Event = z.discriminatedUnion('type', [
  z.object({ type: z.literal('click'), x: z.number() }),
  z.object({ type: z.literal('scroll'), y: z.number() }),
]);
```

## Tipos nominales con .brand

```typescript
const UserId = z.string().brand<'UserId'>();
type UserId = z.infer<typeof UserId>; // string & { __brand: 'UserId' }
```

## Buenas prácticas

- Usa SIEMPRE `z.infer<typeof schema>` en lugar de interfaces manuales duplicadas
- Mantén el esquema como única fuente de verdad del tipo
- Para PATCH, usa `.partial()`: `User.partial()`
- Para selección, usa `.pick()`/`.omit()`