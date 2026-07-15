# Skills Review Session — Julio 2026

> **Fecha**: 2026-07-15
> **Skills afectadas**: `commits`, `changelog-maintenance`
> **Estado**: Todos los puntos completados ✅

---

## Índice

1. [Resumen General](#1-resumen-general)
2. [Punto 1 — Correcciones Ortográficas](#2-punto-1--correcciones-ortográficas)
3. [Punto 2 — Gitmoji Obligatorio](#3-punto-2--gitmoji-obligatorio)
4. [Punto 3 — Referencias MCP](#4-punto-3--referencias-mcp)
5. [Punto 4 — Unificar Idioma a Inglés](#5-punto-4--unificar-idioma-a-inglés)
6. [Punto 5 — Ejemplos Faltantes](#6-punto-5--ejemplos-faltantes)
7. [Punto 6 — Regla 50 vs 72 Caracteres](#7-punto-6--regla-50-vs-72-caracteres)
8. [Punto 7 — Related Tools](#8-punto-7--related-tools)
9. [Punto 8 — Gitmoji a references/](#9-punto-8--gitmoji-a-references)
10. [Punto 9 — changelog-maintenance: Línea truncada + Examples](#10-punto-9--changelog-maintenance-línea-truncada--examples)

---

## 1. Resumen General

Se revisaron y mejoraron dos skills del directorio `skills/`:

### `skills/commits/`

| Archivo | Antes | Después | Diferencia |
|---------|-------|---------|------------|
| `SKILL.md` | ~926 líneas | ~339 líneas | -587 (gitmoji extraído) |
| `references/GITMOJI.md` | — | ~617 líneas | Nuevo |

### `skills/changelog-maintenance/`

| Archivo | Antes | Después | Diferencia |
|---------|-------|---------|------------|
| `SKILL.md` | ~346 líneas | ~437 líneas | +91 (examples añadidos) |

---

## 2. Punto 1 — Correcciones Ortográficas

**Archivo**: `skills/commits/SKILL.md`

| # | Línea | Error | Corrección |
|---|-------|-------|------------|
| 1 | 33 | `<tipo>[ámplito opcional]` | `<tipo>[ámbito opcional]` |
| 2 | 61 | `No usar ponto y coma` | `No usar punto y coma` |
| 3 | 63 | `### Scope (ámplito)` | `### Scope (ámbito)` |
| 4 | 65 | `sección del codebaseaffected` | `sección del codebase affected` |
| 5 | 101 | `actualizar调用 a /api/v2/auth` | `actualizar llamados a /api/v2/auth` |

---

## 3. Punto 2 — Gitmoji Obligatorio

**Decisión**: Gitmoji es **obligatorio** al inicio de la primera línea.

**Formato**:
```
<gitmoji> <tipo>[scope]: <descripción>
```

**Cambios aplicados**:

| Sección | Cambio |
|---------|--------|
| Plantilla de estructura | `<gitmoji>` agregado al inicio del template |
| Reglas de descripción | Nueva regla #1: "Gitmoji required at the beginning" |
| Quick Reference Card | `<gitmoji>` incluido en la card |
| 7 ejemplos existentes | Gitmoji añadido a cada uno (✨, 🐛, ♻️, 💥, 📝, ⬆️, 👷) |
| Anti-Patterns | Ejemplos correctos actualizados con gitmoji |

---

## 4. Punto 3 — Referencias MCP

**Problema**: La skill referenciaba un MCP server `git-auto-commit` con herramienta `git-changes-commit-message` que no existe.

**Corrección**: Actualizado a `auto-commit` con herramienta `auto-commit_git-changes-commit-message`.

**6 lugares corregidos en `commits/SKILL.md`**:

| Línea | Antes | Después |
|-------|-------|---------|
| 3 | `MCP git-auto-commit` | `MCP auto-commit` |
| 15 | `MCP git-auto-commit` | `MCP auto-commit` |
| 19 | `` `git-auto-commit` `` | `` `auto-commit` `` |
| 24 | `` `git-changes-commit-message` `` | `` `auto-commit_git-changes-commit-message` `` |
| 117-120 | `git-changes-commit-message` | `auto-commit_git-changes-commit-message` |
| 275 | `git-auto-commit MCP` | `auto-commit MCP` |

**1 lugar en `changelog-maintenance/SKILL.md`**:

| Línea | Antes | Después |
|-------|-------|---------|
| 111 | `` `git-changes-commit-message` `` | `` `auto-commit_git-changes-commit-message` `` |

---

## 5. Punto 4 — Unificar Idioma a Inglés

Ambos skills se unificaron a **100% inglés**.

### `commits/SKILL.md` — ~30 secciones traducidas

| Sección | Traducción |
|---------|-----------|
| Frontmatter description | `Genera mensajes de commit usando...` → `Generates commit messages using...` |
| When to use this skill | 3 items traducidos de español a inglés |
| MCP Integration | Párrafo completo traducido |
| Required Structure | `Estructura obligatoria`, `cuerpo`, `pie` → inglés |
| Commit Types | Tabla completa con descripciones traducidas |
| Description Rules | 6 reglas traducidas |
| Scope | Párrafo explicativo + `Nombres de módulos...` traducido |
| Body | 4 items traducidos |
| Footer | 3 items traducidos |
| Breaking Changes | `Método 1/2` → `Method 1/2`, `en footer` → `in footer` |
| Step 1, Step 2 | Títulos y contenido traducido |
| 7 Examples | Headers y descripciones traducidos ("Se añade" → "Adds", etc.) |
| Quick Reference Card | Descripciones de tipos en inglés |
| Anti-Patterns | `(evitar)` → `(avoid)` |
| Related Tools | Descripciones traducidas |
| Gitmoji Reference | `Los commits deben empezar...` → `Commits must start...` |

### `changelog-maintenance/SKILL.md` — ~5 fragmentos

| Línea | Traducción |
|-------|-----------|
| 109 | `Generar changelog desde commits` → `Generate changelog from commits` |
| 111 | `Usar la herramienta MCP...` → `Use the MCP tool...` |
| 114, 117, 120 | Comentarios bash: `Últimos 7 días`, `Desde último tag`, `Por rango de fechas` |
| 124 | `Filtrar commits internos...` → `Filter internal commits...` |
| 153 | `Release (ejecutar)` → `Release` |
| 156-165 | Comentarios bash de release traducidos |

---

## 6. Punto 5 — Ejemplos Faltantes

**3 nuevos ejemplos agregados** a `commits/SKILL.md` (Examples 8-10):

| # | Tipo | Gitmoji | Descripción |
|---|------|---------|-------------|
| 8 | `chore` | 🧹 | Remove debug logs and unused imports |
| 9 | `style` | 🎨 | Format codebase with Prettier |
| 10 | `revert` | ⏪️ | Rollback user profile feature (con `Fixes #892`) |

---

## 7. Punto 6 — Regla 50 vs 72 Caracteres

**Decisión (Opción A)**: 50 como recomendación, 72 como hard limit.

**Cambios**:

| Ubicación | Antes | Después |
|-----------|-------|---------|
| Description Rules #2 | `Maximum 50 characters for the first line (excluding the gitmoji)` | `Maximum 50 characters recommended for the first line (hard limit: 72, both excluding the gitmoji)` |
| Step 2 checklist | `Maximum 50 characters in the first line` | `Maximum 50 characters recommended (hard limit: 72) in the first line` |

---

## 8. Punto 7 — Related Tools

**Antes** (referencia circular + solo 3 tools):
```markdown
- **auto-commit MCP**: MCP server that generates commits with AI and gitmoji (this skill integrates it)
- **commitlint**: Validates commits against Conventional Commits
- **commitizen**: Interactive tool for creating commits
```

**Después** (sin referencia circular + 5 herramientas):
```markdown
- **commitlint**: Validates commit messages against Conventional Commits rules. Useful as a git hook to enforce consistency.
- **commitizen**: Interactive CLI that helps craft Conventional Commits with prompts for type, scope, and description.
- **standard-version**: Automates version bumping and changelog generation from Conventional Commits.
- **semantic-release**: Fully automated package publishing — analyzes commits to determine version bumps and generates release notes.
- **husky** + **lint-staged**: Pair used to run commitlint and other validators as git hooks before each commit.
```

---

## 9. Punto 8 — Gitmoji a references/

**Extracción** del JSON de gitmojis (75 entradas) desde `SKILL.md` a un archivo independiente.

**Archivo creado**: `skills/commits/references/GITMOJI.md` (~617 líneas)

Contiene:
- Frontmatter metadata
- JSON completo con los 75 gitmojis (emoji, entity, code, description, name, semver)
- Link a [gitmoji.dev](https://gitmoji.dev)

**SKILL.md actualizado** con tabla resumen de los 12 gitmojis más comunes:

| Emoji | Code | Type | Description |
|-------|------|------|-------------|
| ✨ | `:sparkles:` | feat | Introduce new features |
| 🐛 | `:bug:` | fix | Fix a bug |
| 📝 | `:memo:` | docs | Add or update documentation |
| ♻️ | `:recycle:` | refactor | Refactor code |
| 🎨 | `:art:` | style | Improve structure / format |
| ⚡️ | `:zap:` | perf | Improve performance |
| ✅ | `:white_check_mark:` | test | Add or pass tests |
| 🔧 | `:wrench:` | chore | Add or update configuration |
| 👷 | `:construction_worker:` | ci | Add or update CI build system |
| 🔥 | `:fire:` | - | Remove code or files |
| 💥 | `:boom:` | feat! | Introduce breaking changes |
| ⏪️ | `:rewind:` | revert | Revert changes |

---

## 10. Punto 9 — changelog-maintenance: Línea truncada + Examples

### Línea truncada

**Antes** (Best practices #4):
```
4. **Keep a 
```

**Después**:
```
4. **Keep a deprecation timeline**: announce features before removing them
```

### Examples vacíos

**Example 1**: Basic usage — From commits to changelog
- Muestra 5 commits de ejemplo (feat, fix, docs, chore, refactor)
- Genera CHANGELOG.md filtrando commits internos
- Incluye nota: "Internal commits (refactor, chore, style) are filtered out"

**Example 2**: Advanced usage — Full release workflow
- Pipeline completo en 5 pasos:
  1. `git log` para revisar commits
  2. Generar CHANGELOG.md categorizado (Added, Changed, Deprecated, Removed, Security)
  3. Comandos de release (`npm version`, `git push`, `gh release`)
  4. Release notes user-friendly (RELEASES.md)
  5. Migration guide para breaking changes

---

## Estructura Final de Archivos

```
skills/commits/
├── SKILL.md                  # ~339 líneas (skill principal)
└── references/
    └── GITMOJI.md            # ~617 líneas (lista completa gitmoji)

skills/changelog-maintenance/
└── SKILL.md                  # ~437 líneas (skill principal)
```

---

*Fin del documento — Todos los puntos completados.*
