---
name: release
description: Manage the full release lifecycle for any project: generate user-facing changelogs from git commits, maintain CHANGELOG.md, determine semantic version bumps, execute git tags, and publish GitHub releases. Use when creating or planning a release, writing changelogs, bumping versions, or publishing release notes.
---

# Release Workflow

Guía completa para el ciclo de vida de una release: desde generar un changelog user-friendly a partir de commits hasta publicar en GitHub.

---

## 1. Semantic Versioning

Dado `MAJOR.MINOR.PATCH`:

| Bump | Cuándo | Ejemplo |
|------|--------|---------|
| **MAJOR** | Breaking changes (`BREAKING CHANGE` en commit) | `1.0.0` → `2.0.0` |
| **MINOR** | Nuevas funcionalidades (`feat`) | `1.0.0` → `1.1.0` |
| **PATCH** | Bug fixes, refactors, mejoras menores (`fix`, `refactor`, `chore`, etc.) | `1.0.0` → `1.0.1` |

### Cómo determinar el bump

```bash
# Revisar commits desde el último tag
git log $(git describe --tags --abbrev=0)..HEAD --oneline --no-merges
```

- Si hay `BREAKING CHANGE` → **MAJOR**
- Si hay `feat` → **MINOR**
- Si solo hay `fix`, `refactor`, `chore`, `docs`, `style`, `perf` → **PATCH**

> **⚠️ Sin package.json?** Si el proyecto no tiene `package.json`, omite `npm version` y maneja la versión solo con git tags y CHANGELOG.md.

---

## 2. Generar changelog desde commits

### Analizar commits

```bash
# Últimos 7 días
git log --since="7 days ago" --oneline

# Desde el último tag
git log $(git describe --tags --abbrev=0)..HEAD --oneline

# Por rango de fechas
git log --after="2025-01-01" --before="2025-01-15" --oneline
```

### Transformar a lenguaje user-facing

Los mensajes técnicos de commit deben traducirse a lenguaje que el usuario entienda:

| Commit técnico | Changelog user-facing |
|---|---|
| `feat(api): add paginated users endpoint` | Nuevo endpoint de usuarios con paginación |
| `fix(auth): resolve token refresh race condition` | Corregida condición de carrera en renovación de tokens |
| `refactor(db): extract query builder` | _(se omite: solo refactor interno)_ |
| `chore: bump lodash to 4.17.21` | _(se omite: mantenimiento interno)_ |

**Regla**: filtrar commits internos (`refactor`, `chore`, `style`, `test`) a menos que tengan impacto visible para el usuario.

---

## 3. Formato CHANGELOG.md

Sigue el estándar [Keep a Changelog](https://keepachangelog.com/). Orden reverso-cronológico (última versión arriba).

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [X.Y.Z] - YYYY-MM-DD

### Added
- Nueva funcionalidad A
- Nueva funcionalidad B

### Changed
- Cambios en funcionalidades existentes

### Deprecated
- Funcionalidades que serán eliminadas en el futuro

### Removed
- Funcionalidades eliminadas en esta versión

### Fixed
- Correcciones de bugs

### Security
- Parches de seguridad
```

### Categorización de cambios

| Tipo de commit | Categoría en CHANGELOG |
|---|---|
| `feat` | **Added** |
| `fix` | **Fixed** |
| `perf` | **Changed** |
| `refactor` | _(omitir, o **Changed** si afecta al usuario)_ |
| `docs` | _(omitir, o **Changed** si es documentación pública)_ |
| `style` | _(omitir)_ |
| `chore` | _(omitir)_ |
| `BREAKING CHANGE` | Agregar sub-nota en **Changed** o sección aparte |

### Formato alternativo (user-facing con emojis)

```markdown
## [X.Y.Z] - YYYY-MM-DD

### ✨ Nuevas Funcionalidades
### 🐛 Correcciones
### ♻️ Mejoras
### 🔧 Mantenimiento
```

---

## 4. Workflow completo de release

### Paso 1: Preparar

```bash
git checkout main && git pull
git status
git log $(git describe --tags --abbrev=0)..HEAD --oneline --no-merges
```

### Paso 2: Bump de versión

```bash
# Si tiene package.json:
npm version patch   # o minor, major
# O editar package.json manualmente

# Si no tiene package.json: la versión se maneja solo con git tags
```

### Paso 3: Actualizar CHANGELOG.md

Añadir entrada al inicio siguiendo el formato de la sección 3. Agrupar cambios por categoría.

### Paso 4: Commit de release

```bash
git add package.json CHANGELOG.md
git commit -m "🔖 chore(release): bump version to X.Y.Z"
```

### Paso 5: Crear tag

```bash
git tag -a vX.Y.Z -m "🔖 vX.Y.Z"
```

### Paso 6: Publicar release en GitHub

```bash
gh release create vX.Y.Z \
  --title "vX.Y.Z" \
  --notes "Ver CHANGELOG.md para detalles" \
  --target main
```

Para pre-release:

```bash
gh release create vX.Y.Z \
  --title "vX.Y.Z (beta)" \
  --notes "Versión preliminar" \
  --prerelease
```

### Paso 7: Post-release (push)

```bash
git push origin vX.Y.Z
git push origin main
```

---

## 5. Release Notes (user-friendly)

Para comunicar cambios a usuarios no-técnicos:

```markdown
# Release Notes vX.Y.Z
**Released**: YYYY-MM-DD

## 🎉 What's New

### Feature Title
Descripción del beneficio para el usuario.

## ✨ Improvements

- Mejora 1
- Mejora 2

## 🐛 Bug Fixes

- Corrección 1
- Corrección 2

## ⚠️ Breaking Changes

- Cambio que requiere acción del usuario
  - **Migración**: instrucciones
```

---

## 6. Migration Guide (breaking changes)

Cuando hay breaking changes, crear guía de migración:

```markdown
# Migration Guide: v1.x → v2.0

## Authentication Method Changed

**Before** (v1.x):
```javascript
// código antiguo
```

**After** (v2.0):
```javascript
// código nuevo
```

## Deprecation Timeline

- v2.0: feature marcada como deprecated
- v2.1: warnings de deprecación
- v2.2: feature eliminada
```
```

---

## 7. Ejemplo completo

```bash
# 1. Revisar cambios
git log v0.2.1..HEAD --oneline

# 2. Bump (ej: hay feat → MINOR → 0.3.0)
npm version minor
# O editar package.json manualmente

# 3. Actualizar CHANGELOG.md con los cambios categorizados

# 4. Commit
git add package.json CHANGELOG.md
git commit -m "🔖 chore(release): bump version to 0.3.0"

# 5. Tag
git tag -a v0.3.0 -m "🔖 v0.3.0"

# 6. Release
gh release create v0.3.0 --title "v0.3.0" --notes "See CHANGELOG.md" --target main

# 7. Push
git push origin main && git push origin v0.3.0
```

---

## 8. Output files

| Archivo | Propósito |
|---|---|
| `CHANGELOG.md` | Histórico developer-facing (Keep a Changelog) |
| `package.json` | Versión actualizada (si aplica) |
| `RELEASE_NOTES.md` | Notas user-facing para la release |
| `docs/migration-vX-to-vY.md` | Guía de migración (breaking changes) |

---

## 9. Constraints

### Required (MUST)
1. **Orden reverso-cronológico**: últma versión arriba
2. **Incluir fechas**: ISO 8601 (`YYYY-MM-DD`)
3. **Categorizar entradas**: Added, Changed, Fixed, etc.
4. **Preguntar al usuario** antes de hacer push o publicar release
5. **Confirmar el bump** con el usuario antes de ejecutar
6. **Siempre revisar `git status`** antes de cualquier operación
7. Si el proyecto tiene `package.json`, actualizarlo; si no, solo git tags
8. Tags deben seguir formato `vX.Y.Z`

### Prohibited (MUST NOT)
1. **No copiar commits literales** al CHANGELOG — escribir desde perspectiva del usuario
2. **No ser vago**: "Bug fixes", "Performance improvements" no son específicos
3. **No hacer push sin preguntar**
4. **No crear releases sin confirmación del usuario**
5. **No modificar archivos fuera de** `package.json` y `CHANGELOG.md` en un commit de release

---

## 10. Best practices

1. **Keep a Changelog**: seguir el formato estándar
2. **Semantic Versioning**: gestión de versiones consistente
3. **Breaking Changes**: siempre proveer guía de migración
4. **Deprecation timeline**: anunciar features antes de eliminarlas
5. **Commit filtering**: filtrar refactor/chore/style del changelog público
6. **Escribir para el usuario**: lenguaje claro, beneficios, no detalles técnicos internos

---

## References

- [Keep a Changelog](https://keepachangelog.com/)
- [Semantic Versioning](https://semver.org/)
