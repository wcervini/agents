---
description: Reglas de manejo de errores y mensajes con Zod
globs:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.astro"
---

# Zod Error Handling Rules

## MUST DO

- Usar `z.treeifyError(err)` para estructuras anidadas y `z.flattenError(err)` para esquemas planos (Zod v4)
- Acceder a `issue.path` para saber qué campo falló
- Proporcionar mensajes claros y en el idioma del usuario final
- Mapear `fieldErrors` a cada input en formularios
- Usar `errorMap` o mensajes inline para personalizar errores

## MUST NOT DO

- Usar `.format()`/`.flatten()` en Zod v4 (deprecados)
- Exponer errores internos (stack traces) al cliente
- Devolver el `ZodError` completo al usuario final sin formatear
- Ignorar el resultado de `safeParse` sin manejar el caso `success: false`

## Patrones

### Capturar ZodError

```typescript
import { z } from 'zod';

try {
  schema.parse(input);
} catch (err) {
  if (err instanceof z.ZodError) {
    err.issues.forEach((issue) => {
      console.log(issue.path, issue.message, issue.code);
    });
  }
}
```

### Formatear para la API (v4)

```typescript
const result = schema.safeParse(input);
if (!result.success) {
  const flat = z.flattenError(result.error);
  // { formErrors: [], fieldErrors: { email: ['Email inválido'] } }
  return res.status(400).json({ errors: flat });
}
```

### Mensajes personalizados

```typescript
const User = z.object({
  name: z.string().min(2, 'El nombre debe tener al menos 2 caracteres'),
  email: z.string().email('Email inválido'),
});
```