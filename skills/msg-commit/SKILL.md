---
name: msg-commit
description: "Agrupa archivos modificados por intención y sugiere commits semánticos con gitmoji. No ejecuta nada, solo aconseja."
risk: low
source: community
date_added: "2026-08-04"
---

# msg-commit

> Skill para agrupar archivos modificados por intención y generar sugerencias de commits semánticos con gitmoji.

## Workflow

Al invocar esta skill, sigue estos pasos **sin ejecutar ningún comando git** — solo analiza y sugiere:

### 1. Inspeccionar el estado actual

Ejecuta para entender qué cambió:
- `git status --short`
- `git diff --stat`
- `git diff`
- `git log --oneline -10`

### 2. Revisar archivos sensibles

Antes de agrupar, verifica si hay archivos como `.env`, tokens, credenciales. Si aparecen, **detente y avisa al usuario**.

### 3. Agrupar por intención

Identifica grupos de archivos relacionados por propósito:

| Intención | Descripción |
|-----------|-------------|
| Misma feature | Archivos que implementan una misma funcionalidad nueva |
| Mismo fix | Archivos que corrigen un mismo bug |
| Mismo refactor | Archivos reestructurados sin cambio de comportamiento |
| Misma docs | Solo cambios de documentación |
| Mismo chore | Config, tooling, dependencias |
| Mismo style | Formato, estilo de código |
| Misma perf | Optimizaciones de rendimiento |
| Mismos tests | Solo archivos de test |

No mezcles cambios de diferente intención en el mismo grupo.

### 4. Sugerir cada grupo

Para cada grupo, presenta al usuario:

```
── Grupo 1 ──────────────────────────
  git add <archivo1> <archivo2> ...
  Tipo:    <feat|fix|refactor|docs|chore|style|perf|test|ci|build>
  Scope:   <módulo o área afectada>
  Mensaje: <gitmoji> <tipo>(<scope>): <descripción>
```

Determina el **tipo** según la intención del grupo y el **scope** según el módulo o área (ej: `auth`, `api`, `ui`, `db`, `config`, `i18n`, `core`).

### 5. No ejecutar nada

No hagas `git add`, `git commit`, `git push`, ni ninguna operación que modifique el repositorio. Tu función es **solo sugerir**.

## Reglas del mensaje de commit

- Formato: `<gitmoji> <tipo>(<alcance>): <descripcion>`
- Máximo 72 caracteres
- Imperativo, sin punto final
- No mezcles cambios no relacionados en el mismo commit

## Gitmoji reference

| Gitmoji | Tipo | Descripción |
|---------|------|-------------|
| ✨ | feat | Nueva feature |
| 🐛 | fix | Bug fix |
| ♻️ | refactor | Refactor |
| 📝 | docs | Documentación |
| 🎨 | style | Formato, estilo |
| ⚡️ | perf | Performance |
| ✅ | test | Tests |
| 🔧 | chore | Config, tooling |
| 👷 | ci | CI/CD |
| ⬆️ | build | Dependencias |
| 💥 | feat! | Breaking change |
| ⏪️ | revert | Revert |
| 🔥 | remove | Eliminar código |
| 🚚 | rename | Renombrar o mover |

## Identidad del commit (referencia)

Si el usuario decide commitear, la identidad a usar es:

```
git -c user.name="opencode-ai" -c user.email="wcervini+opencode-ai@gmail.com" -c user.signingkey="$HOME/.ssh/opencode_signing.pub" commit -S -m "<mensaje>"
```
