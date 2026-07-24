---
name: commits
description: Genera mensajes de commit con gitmoji y Conventional Commits usando un agente.
metadata:
  tags: git, conventional-commits, gitmoji, mcp, version-control
  platforms: OpenCode, Claude, Gemini, ChatGPT, VSCode
---

# Conventional Commits Architect con Gitmoji

## Cuándo usarlo

- **Antes de un commit**: Para generar mensajes de commit estandarizados.
- **Estandarización del flujo Git**: Commits consistentes con gitmoji + Conventional Commits.
- **Agent workflow**: Usa un agente para generar mensajes de commit con IA.

## Agente

Usa un agente integrado (auto-commit) que:
1. Analiza cambios con `git diff` o contexto proporcionado
2. Genera mensajes con gitmoji
3. Proporciona el mensaje al usuario para confirmación
4. El usuario verifica y ejecuta `git commit` manualmente

## Especificación

### Estructura obligatoria

```
<gitmoji> <tipo>(<ámbito>): <descripción>

[cuerpo opcional]

[pie opcional]
```

### Tipos de commit

| Tipo       | Descripción                                                    |
| ---------- | -------------------------------------------------------------- |
| `feat`     | Nueva característica para el usuario                           |
| `fix`      | Corrección de errores                                          |
| `docs`     | Solo documentación                                             |
| `style`    | Cambios que no afectan el significado (formato, espacios, etc) |
| `refactor` | Refactorización que no agrega feature ni corrige bug           |
| `perf`     | Cambios que mejoran rendimiento                                |
| `test`     | Añadir o corregir pruebas                                      |
| `build`    | Cambios en sistema de compilación o dependencias               |
| `ci`       | Cambios en configuración de CI                                 |
| `chore`    | Otros cambios que no modifican src o tests                     |

### Reglas de descripción

1. **Gitmoji obligatorio** al inicio (✨, 🐛, 📝, etc.)
2. **Máximo 50 caracteres** recomendado (límite: 72, sin contar el gitmoji)
3. **Sin punto al final**
4. **Modo imperativo**: "add feature", no "added feature" ni "adds feature"
5. **Sin mayúscula inicial**

### Ámbito (obligatorio)

Identifica la sección de la base de código afectada:

```
✨ feat(auth): add OAuth2 login
🐛 fix(api): resolve race condition in user endpoint
📝 docs(readme): update installation instructions
```

**Ámbitos comunes**: `api`, `ui`, `db`, `auth`, `config`, `build`, `deps`, `docs`, `tests` o nombres específicos del proyecto.

### Cuerpo

- Separar con línea en blanco después de la descripción
- Máximo 72 caracteres por línea
- Explicar **qué** y **por qué**, no **cómo**
- Usar viñetas con `-` para múltiples líneas

### Pie

- Separar con línea en blanco después del cuerpo
- Referenciar issues/PRs: `Closes #123`, `Fixes #456`, `Refs #789`
- Breaking changes: `BREAKING CHANGE: descripción`

### Breaking Changes

**Método 1** (en el pie):

```
BREAKING CHANGE: la API de autenticación ahora requiere JWT.
Migración: actualizar llamadas a /api/v2/auth.
```

**Método 2** (en tipo/ámbito):

```
💥 feat(api)!: remove support for Node 16
```

---

## Instrucciones

### Prerrequisitos

- ✅ Verificar que existe el repo: `git rev-parse --git-dir 2>/dev/null`

### Paso 1: Generar mensaje con MCP

1. **Analiza los cambios** con `git diff --staged` o el contexto proporcionado.
2. **Propón el mensaje al usuario** mostrando el formato completo (gitmoji, tipo, ámbito, descripción).
3. **Espera confirmación explícita** antes de ejecutar. Si el usuario acepta, ejecuta:

```
auto-commit_git-changes-commit-message
```

> ⚠️ **Nunca ejecutes el MCP auto-commit sin confirmación explícita del usuario**, incluso si el mensaje parece correcto.

### Paso 2: Verificar resultado

El mensaje debe cumplir:
- ✅ Gitmoji al inicio
- ✅ Tipo correcto (feat, fix, etc.)
- ✅ Ámbito presente
- ✅ Máximo 72 caracteres en la primera línea
- ✅ Sin punto final
- ✅ Modo imperativo

---

## Ejemplos

### Ejemplo 1: Nueva funcionalidad

**Diff**: Añade endpoint `/api/users` con paginación.

```
✨ feat(api): add paginated users endpoint

- GET /api/users?page=1&limit=20
- Returns { data, pagination }
- Validates query parameters
- Includes rate limiting

Closes #234
```

### Ejemplo 2: Corrección de bug

**Diff**: Corrige fuga de memoria en el worker de trabajos.

```
🐛 fix(worker): resolve memory leak in job processor

The processor was not releasing event listeners on completion.
Now properly cleans up all listeners after each job.

Fixes #567
```

### Ejemplo 3: Breaking change

**Diff**: Elimina API v1 legacy.

```
💥 feat(api): remove deprecated v1 API endpoints

BREAKING CHANGE: All /api/v1/* endpoints removed.
Migration guide: see docs/migration-v2.md

Closes #890
```

### Ejemplo 4: Revertir cambios

**Diff**: Revierte un deploy fallido.

```
⏪️ revert: rollback user profile feature

The deployment caused database connection timeouts
under load. Reverting to the previous stable version.

Fixes #892
```

---

## Referencia Rápida

```
┌────────────────────────────────────────────────────────┐
│  ✨ <tipo>(<ámbito>): <descripción>                    │
│                                                        │
│  [cuerpo opcional]                                     │
│                                                        │
│  [pie opcional: Closes #N, BREAKING CHANGE: ...]       │
└────────────────────────────────────────────────────────┘

✨ feat(scope)     → Nueva funcionalidad
🐛 fix(scope)     → Corrección de bug
📝 docs(scope)    → Documentación
🎨 style(scope)   → Formato
♻️ refactor(scope) → Refactorización
⚡️ perf(scope)    → Rendimiento
✅ test(scope)    → Tests
⬆️ build(scope)   → Dependencias
👷 ci(scope)      → CI/CD
🧹 chore(scope)   → Mantenimiento
⏪️ revert(scope)  → Reversión

Commits atómicos: un tipo, un propósito. Ámbito obligatorio.
```

---

## Anti-Patrones (evitar)

❌ `Fixed the bug` — no sigue formato
❌ `✨ feat: add feature` — falta ámbito
❌ `feat: Add new feature` — falta gitmoji, mayúscula incorrecta

✅ `✨ feat(api): add JWT refresh token rotation`
✅ `🐛 fix(api): resolve race condition in user endpoint`
✅ `📝 docs(readme): update API documentation`

---

## Referencia de Gitmojis

Ver lista completa en [references/GITMOJI.md](references/GITMOJI.md).

| Emoji | Tipo    | Descripción               |
|-------|---------|---------------------------|
| ✨     | feat    | Nueva funcionalidad       |
| 🐛     | fix     | Corrección de bug         |
| 📝     | docs    | Documentación             |
| ♻️     | refactor| Refactorización           |
| 🎨     | style   | Formato                   |
| ⚡️    | perf    | Rendimiento               |
| ✅     | test    | Tests                     |
| 🔧     | chore   | Configuración             |
| 👷     | ci      | CI/CD                     |
| 💥     | feat!   | Breaking change           |
| ⏪️    | revert  | Reversión                 |

Más info en [gitmoji.dev](https://gitmoji.dev)
