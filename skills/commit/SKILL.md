---
name: cc
description: Analiza los cambios en el área de preparación (staged) a través del git diff para generar un único mensaje de commit perfecto bajo la especificación estricta de Conventional Commits.
metadata:
  tags: git, conventional-commits, frontend-architect, version-control
  platforms: OpenCode, Claude, Gemini, ChatGPT
---

# Conventional Commits Architect

## When to use this skill

- **Before committing**: Cuando necesitas redactar o automatizar el mensaje de confirmación para tus cambios preparados.
- **Git workflow standardization**: Para asegurar que todo el historial de commits del repositorio mantenga una estructura limpia, profesional y homogénea.
- **Automated tooling**: Para integrar con herramientas que generan changelogs automáticos desde commits.

## Specification

### Estructura obligatoria

```
<tipo>[ámplito opcional]: <descripción>

[cuerpo opcional]

[pie opcional]
```

### Tipos de commit

| Tipo       | Descripción                                                                |
| ---------- | -------------------------------------------------------------------------- |
| `feat`     | Nueva funcionalidad para el usuario                                        |
| `fix`      | Corrección de un bug que afecta al usuario                                 |
| `docs`     | Cambios solo en documentación                                              |
| `style`    | Cambios que no afectan el significado del código (formato, espacios, etc.) |
| `refactor` | Refactorización del código que ni corrige un bug ni agrega feature         |
| `perf`     | Cambios que mejoran el rendimiento                                         |
| `test`     | Añadir o corregir tests                                                    |
| `build`    | Cambios que afectan al sistema de build o dependencias                     |
| `ci`       | Cambios en archivos de configuración de CI                                 |
| `chore`    | Otros cambios que no modifican src ni archivos de test                     |

### Reglas para la descripción

1. **Máximo 50 caracteres** para la primera línea
2. **Sin punto final**
3. **Usar imperativo**: "add feature" no "added feature" / "adds feature"
4. **No capitalizar** la primera letra
5. **No usar ponto y coma** al final

### Scope (ámplito)

El scope es opcional y identifica la sección del codebaseaffected:

```
feat(auth): add OAuth2 login
fix(api): resolve race condition in user endpoint
docs(readme): update installation instructions
```

**Scopes comunes**:

- `api`, `ui`, `db`, `auth`, `config`, `build`, `deps`, `docs`, `tests`
- Nombres de módulos específicos del proyecto

### Body (cuerpo)

- Separar con línea en blanco después de la descripción
- Máximo 72 caracteres por línea
- Explicar **qué** y **por qué**, no **cómo**
- Usar bullet points con `-` para múltiples líneas

### Footer (pie)

- Separar con línea en blanco después del body
- Referenciar issues y PRs:
  - `Closes #123`
  - `Fixes #456`
  - `Refs #789`
- Listar cambios que rompen compatibilidad:
  - `BREAKING CHANGE: remove deprecated API`

### Breaking Changes

**Método 1** (en footer):

```
BREAKING CHANGE: la API de autenticación ahora requiere token JWT.
Migration: actualizar调用 a /api/v2/auth.
```

**Método 2** (en tipo/scope):

```
feat!: remove support for Node 16
```

---

## Instructions

### Step 1: Obtener el diff

Ejecuta `git diff --staged` para ver los cambios en el área de preparación. Si el entorno lo permite, obtén el diff de forma autónoma.

### Step 2: Analizar los cambios

Identifica:

1. **Qué archivos cambiaron**
2. **Qué tipo de cambio predomina** (feat, fix, refactor, etc.)
3. **Qué sección del código fue afectada** (scope)
4. **Si hay breaking changes**
5. **Si hay issues/PRs relacionados**

### Step 3: Redactar el commit

#### Reglas de prioridad

1. Si hay múltiples tipos, usar el **más específico**:
   - Un refactor que mejora rendimiento → `perf` (no `refactor`)
   - Un fix de bug en tests → `test` (no `fix`)

2. Si hay múltiples scopes, usar el **más importante** o remover scope

3. Si los cambios son muy diversos, hacer commits separados cuando tenga sentido

### Step 4: Verificar el mensaje

- ¿La descripción tiene máximo 50 caracteres?
- ¿Usa imperativo?
- ¿No tiene punto final?
- ¿El scope es consistente con el proyecto?

---

## Examples

### Example 1: Nueva funcionalidad

**Diff**: Se añade endpoint `/api/users` con paginación.

```
feat(api): add paginated users endpoint

- GET /api/users?page=1&limit=20
- Returns { data, pagination }
- Validates query parameters
- Includes rate limiting

Closes #234
```

### Example 2: Bug fix

**Diff**: Se corrige memory leak en el worker de jobs.

```
fix(worker): resolve memory leak in job processor

The processor was not releasing event listeners on completion.
Now properly cleans up all listeners after each job.

Fixes #567
```

### Example 3: Refactorización

**Diff**: Se extrae lógica de autenticación a un módulo dedicado.

```
refactor(auth): extract auth logic to dedicated module

- Move token validation from controller to AuthService
- Add refresh token rotation
- Reduce cognitive complexity by 40%
```

### Example 4: Breaking change

**Diff**: Se elimina API legacy de v1.

```
feat!: remove deprecated v1 API endpoints

BREAKING CHANGE: All /api/v1/* endpoints removed.
Migration guide: see docs/migration-v2.md

Closes #890
```

### Example 5: Solo documentación

**Diff**: Se actualiza README con nuevas instrucciones de instalación.

```
docs(readme): update installation instructions

- Add Docker setup steps
- Include environment variables reference
- Fix outdated npm commands
```

### Example 6: Cambios de dependencias

**Diff**: Se actualiza Express de 4.x a 5.x.

```
build(deps): upgrade Express to v5

- Migrate to async handlers
- Update middleware signatures
- Required for Node 20+ compatibility
```

### Example 7: Corrección de CI

**Diff**: Se corrige pipeline de GitHub Actions.

```
ci: fix test runner timeout

- Increase timeout from 2min to 5min for integration tests
- Add retry logic for flaky tests
- Update Node version matrix
```

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────┐
│  tipo(scope): descripción                                │
│                                                         │
│  [cuerpo opcional]                                      │
│                                                         │
│  [footer opcional: Closes #N, BREAKING CHANGE: ...]     │
└─────────────────────────────────────────────────────────┘

feat     → Nueva feature
fix      → Bug fix
docs     → Documentación
style    → Formato (no lógica)
refactor → Reestructurar código
perf     → Rendimiento
test     → Tests
build    → Build/dependencias
ci       → CI/CD
chore    → Mantenimiento general

Commits atómicos: un tipo, un propósito
```

---

## Anti-Patterns (evitar)

❌ `Fixed the bug`
❌ `Updated code`
❌ `WIP`
❌ `asdasdasd`
❌ `feat: Add new feature for the user`
❌ `fix: Fixed critical issue in the system.`

✅ `feat(auth): add JWT refresh token rotation`
✅ `fix(api): resolve race condition in user endpoint`
✅ `docs: update API documentation`

---

## Related Tools

- **commitlint**: Valida commits contra Conventional Commits
- **standard-version**: Genera changelog desde commits
- **commitizen**: Herramienta interactiva para crear commits
- **semantic-release**: Publica y versiona automáticamente
