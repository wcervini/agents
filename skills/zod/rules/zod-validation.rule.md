---
description: Reglas de diseño de esquemas y parsing con Zod
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.astro"
---

# Zod Validation Rules

## MUST DO

- Usar `z.infer<typeof schema>` para tipos — nunca duplicar interfaces manuales
- Usar `safeParse`/`safeParseAsync` en el borde de la API (bodies, queries, forms)
- Usar `.refine()`/`.superRefine()` para validaciones de lógica de negocio complejas
- Proporcionar mensajes de error claros y personalizados
- Validar en el borde de la API — nunca confiar en datos del cliente
- En Astro, importar `z` desde `astro/zod` (NO del paquete `zod`)
- En proyectos con Drizzle, derivar los schemas con `drizzle-zod` (`createInsertSchema`/`createSelectSchema`) en vez de `z.object` manual
- Usar `z.coerce.date()` para fechas que llegan como string
- Usar `z.discriminatedUnion` cuando haya un campo discriminador literal
- Usar `z.treeifyError`/`z.flattenError` (v4) en lugar de `.format()`/`.flatten()`
- Verificar APIs dudosas en zod.dev o vía Context MCP

## MUST NOT DO

- Usar `any` — usa `unknown` y estrecha con el esquema
- Duplicar interfaces manuales cuando `z.infer` ya genera el tipo
- Usar `.parse()` en flujos donde el fallo debe manejarse con gracia (usa `safeParse`)
- Inventar métodos de validación/transformación sin verificar en zod.dev
- Importar `z` desde `zod` en Astro (usar `astro/zod`)
- Escribir `z.object` manual para tablas Drizzle existentes — deriva con `drizzle-zod`
- Usar `.format()`/`.flatten()` en Zod v4 (deprecados)
- Confiar en datos sin validar

## Patrones

### Esquema + inferencia

```typescript
import { z } from 'zod';

const User = z.object({
  id: z.number(),
  name: z.string().min(2).max(50),
  email: z.string().email(),
  role: z.enum(['admin', 'user']).default('user'),
});

type User = z.infer<typeof User>;
```

### safeParse en el borde

```typescript
const result = User.safeParse(body);
if (!result.success) {
  return res.status(400).json({ errors: z.flattenError(result.error) });
}
const user = result.data;
```

### Refinamiento de negocio

```typescript
const Password = z.string()
  .min(8, 'Mínimo 8 caracteres')
  .superRefine((val, ctx) => {
    if (val === val.toLowerCase()) {
      ctx.addIssue({ code: z.ZodIssueCode.custom, message: 'Debe tener una mayúscula' });
    }
  });
```