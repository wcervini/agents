# Manejo de errores con Zod

Cómo capturar, formatear y personalizar los errores de validación.

## ZodError

Cuando `parse` falla lanza un `ZodError` con un array de `issues`:

```typescript
import { z } from 'zod';

const Player = z.object({
  username: z.string(),
  xp: z.number(),
});

try {
  Player.parse({ username: 42, xp: '100' });
} catch (err) {
  if (err instanceof z.ZodError) {
    err.issues;
    /* [
      { code: 'invalid_type', path: ['username'], message: 'Invalid input: expected string', expected: 'string', received: 'number' },
      { code: 'invalid_type', path: ['xp'], message: 'Invalid input: expected number', expected: 'number', received: 'string' },
    ] */
  }
}
```

Cada `ZodIssue` tiene:
- `code` — `ZodIssueCode` (`invalid_type`, `invalid_string`, `too_big`, `too_small`, `custom`, `unrecognized_keys`, `invalid_union`, `invalid_enum_value`, ...)
- `path` — `(string | number)[]` (p.ej. `['user', 'email']`)
- `message` — mensaje legible
- campos específicos según el código (`expected`, `received`, `minimum`, `maximum`, ...)

## Formato de errores (Zod v4)

En Zod 4 los métodos `.format()`/`.flatten()` del `ZodError` están **deprecados**. Usa las funciones standalone:

```typescript
import { z } from 'zod';

const result = schema.safeParse(input);

if (!result.success) {
  // Árbol anidado que refleja el esquema
  const tree = z.treeifyError(result.error);
  // { user: { email: { _errors: ['Invalid email'] } } }

  // Plano: { formErrors: string[], fieldErrors: { [campo]: string[] } }
  const flat = z.flattenError(result.error);
  // { formErrors: [], fieldErrors: { email: ['Invalid email'] } }

  // Formato anidado estilo v3
  const formatted = z.formatError(result.error);
}
```

- `z.treeifyError` → útil para estructuras anidadas (usa `?.` para acceder).
- `z.flattenError` → para la mayoría de esquemas planos; `formErrors` = errores de nivel raíz (`path: []`), `fieldErrors` = errores por campo.

### Con mapper personalizado

```typescript
const flat = z.flattenError(result.error, (issue) => ({
  message: issue.message,
  errorCode: issue.code,
}));
// { formErrors: [], fieldErrors: { name: [{ message, errorCode }] } }
```

## errorMap — mensajes personalizados

### Por esquema (opción de parse)

```typescript
const customErrorMap: z.ZodErrorMap = (error, ctx) => {
  switch (error.code) {
    case z.ZodIssueCode.invalid_type:
      if (error.expected === 'string') {
        return { message: 'Esto no es un string!' };
      }
      break;
    case z.ZodIssueCode.custom:
      const params = error.params || {};
      if (params.myField) {
        return { message: `Entrada inválida: ${params.myField}` };
      }
      break;
  }
  return { message: ctx.defaultError };
};

z.string().parse(12, { errorMap: customErrorMap });
```

### Global

```typescript
z.setErrorMap(customErrorMap);
```

### Mensajes inline (más común)

```typescript
const User = z.object({
  name: z.string().min(2, 'El nombre debe tener al menos 2 caracteres'),
  email: z.string().email('Email inválido'),
  age: z.number().min(18, 'Debes ser mayor de edad'),
});
```

## superRefine — issues múltiples

```typescript
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
```

## Buenas prácticas

- Mensajes claros y en el idioma del usuario final
- Usa `z.treeifyError`/`z.flattenError` (v4) en lugar de `.format()`/`.flatten()`
- Para formularios, mapea `fieldErrors` a cada input
- No expongas errores internos (stack traces) al cliente