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

### Step 4: update the version:
- before commit update version on Package.json and include on the commit
- Create a git tag version
- Update de CHANGELOG.md
- Manage de Releases: more info on **https://docs.github.com/es/repositories/releasing-projects-on-github/managing-releases-in-a-repository**



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


## LAst Notes Use Emojis on the Commit Scope
samples:
{
  "gitmojis": [
    {
      "emoji": "🎨",
      "entity": "&#x1f3a8;",
      "code": ":art:",
      "description": "Improve structure / format of the code.",
      "name": "art",
      "semver": null
    },
    {
      "emoji": "⚡️",
      "entity": "&#x26a1;",
      "code": ":zap:",
      "description": "Improve performance.",
      "name": "zap",
      "semver": "patch"
    },
    {
      "emoji": "🔥",
      "entity": "&#x1f525;",
      "code": ":fire:",
      "description": "Remove code or files.",
      "name": "fire",
      "semver": null
    },
    {
      "emoji": "🐛",
      "entity": "&#x1f41b;",
      "code": ":bug:",
      "description": "Fix a bug.",
      "name": "bug",
      "semver": "patch"
    },
    {
      "emoji": "🚑️",
      "entity": "&#128657;",
      "code": ":ambulance:",
      "description": "Critical hotfix.",
      "name": "ambulance",
      "semver": "patch"
    },
    {
      "emoji": "✨",
      "entity": "&#x2728;",
      "code": ":sparkles:",
      "description": "Introduce new features.",
      "name": "sparkles",
      "semver": "minor"
    },
    {
      "emoji": "📝",
      "entity": "&#x1f4dd;",
      "code": ":memo:",
      "description": "Add or update documentation.",
      "name": "memo",
      "semver": null
    },
    {
      "emoji": "🚀",
      "entity": "&#x1f680;",
      "code": ":rocket:",
      "description": "Deploy stuff.",
      "name": "rocket",
      "semver": null
    },
    {
      "emoji": "💄",
      "entity": "&#ff99cc;",
      "code": ":lipstick:",
      "description": "Add or update the UI and style files.",
      "name": "lipstick",
      "semver": "patch"
    },
    {
      "emoji": "🎉",
      "entity": "&#127881;",
      "code": ":tada:",
      "description": "Begin a project.",
      "name": "tada",
      "semver": null
    },
    {
      "emoji": "✅",
      "entity": "&#x2705;",
      "code": ":white_check_mark:",
      "description": "Add, update, or pass tests.",
      "name": "white-check-mark",
      "semver": null
    },
    {
      "emoji": "🔒️",
      "entity": "&#x1f512;",
      "code": ":lock:",
      "description": "Fix security or privacy issues.",
      "name": "lock",
      "semver": "patch"
    },
    {
      "emoji": "🔐",
      "entity": "&#x1f510;",
      "code": ":closed_lock_with_key:",
      "description": "Add or update secrets.",
      "name": "closed-lock-with-key",
      "semver": null
    },
    {
      "emoji": "🔖",
      "entity": "&#x1f516;",
      "code": ":bookmark:",
      "description": "Release / Version tags.",
      "name": "bookmark",
      "semver": null
    },
    {
      "emoji": "🚨",
      "entity": "&#x1f6a8;",
      "code": ":rotating_light:",
      "description": "Fix compiler / linter warnings.",
      "name": "rotating-light",
      "semver": null
    },
    {
      "emoji": "🚧",
      "entity": "&#x1f6a7;",
      "code": ":construction:",
      "description": "Work in progress.",
      "name": "construction",
      "semver": null
    },
    {
      "emoji": "💚",
      "entity": "&#x1f49a;",
      "code": ":green_heart:",
      "description": "Fix CI Build.",
      "name": "green-heart",
      "semver": null
    },
    {
      "emoji": "⬇️",
      "entity": "⬇️",
      "code": ":arrow_down:",
      "description": "Downgrade dependencies.",
      "name": "arrow-down",
      "semver": "patch"
    },
    {
      "emoji": "⬆️",
      "entity": "⬆️",
      "code": ":arrow_up:",
      "description": "Upgrade dependencies.",
      "name": "arrow-up",
      "semver": "patch"
    },
    {
      "emoji": "📌",
      "entity": "&#x1F4CC;",
      "code": ":pushpin:",
      "description": "Pin dependencies to specific versions.",
      "name": "pushpin",
      "semver": "patch"
    },
    {
      "emoji": "👷",
      "entity": "&#x1f477;",
      "code": ":construction_worker:",
      "description": "Add or update CI build system.",
      "name": "construction-worker",
      "semver": null
    },
    {
      "emoji": "📈",
      "entity": "&#x1F4C8;",
      "code": ":chart_with_upwards_trend:",
      "description": "Add or update analytics or track code.",
      "name": "chart-with-upwards-trend",
      "semver": "patch"
    },
    {
      "emoji": "♻️",
      "entity": "&#x267b;",
      "code": ":recycle:",
      "description": "Refactor code.",
      "name": "recycle",
      "semver": null
    },
    {
      "emoji": "➕",
      "entity": "&#10133;",
      "code": ":heavy_plus_sign:",
      "description": "Add a dependency.",
      "name": "heavy-plus-sign",
      "semver": "patch"
    },
    {
      "emoji": "➖",
      "entity": "&#10134;",
      "code": ":heavy_minus_sign:",
      "description": "Remove a dependency.",
      "name": "heavy-minus-sign",
      "semver": "patch"
    },
    {
      "emoji": "🔧",
      "entity": "&#x1f527;",
      "code": ":wrench:",
      "description": "Add or update configuration files.",
      "name": "wrench",
      "semver": "patch"
    },
    {
      "emoji": "🔨",
      "entity": "&#128296;",
      "code": ":hammer:",
      "description": "Add or update development scripts.",
      "name": "hammer",
      "semver": null
    },
    {
      "emoji": "🌐",
      "entity": "&#127760;",
      "code": ":globe_with_meridians:",
      "description": "Internationalization and localization.",
      "name": "globe-with-meridians",
      "semver": "patch"
    },
    {
      "emoji": "✏️",
      "entity": "&#59161;",
      "code": ":pencil2:",
      "description": "Fix typos.",
      "name": "pencil2",
      "semver": "patch"
    },
    {
      "emoji": "💩",
      "entity": "&#58613;",
      "code": ":poop:",
      "description": "Write bad code that needs to be improved.",
      "name": "poop",
      "semver": null
    },
    {
      "emoji": "⏪️",
      "entity": "&#9194;",
      "code": ":rewind:",
      "description": "Revert changes.",
      "name": "rewind",
      "semver": "patch"
    },
    {
      "emoji": "🔀",
      "entity": "&#128256;",
      "code": ":twisted_rightwards_arrows:",
      "description": "Merge branches.",
      "name": "twisted-rightwards-arrows",
      "semver": null
    },
    {
      "emoji": "📦️",
      "entity": "&#1F4E6;",
      "code": ":package:",
      "description": "Add or update compiled files or packages.",
      "name": "package",
      "semver": "patch"
    },
    {
      "emoji": "👽️",
      "entity": "&#1F47D;",
      "code": ":alien:",
      "description": "Update code due to external API changes.",
      "name": "alien",
      "semver": "patch"
    },
    {
      "emoji": "🚚",
      "entity": "&#1F69A;",
      "code": ":truck:",
      "description": "Move or rename resources (e.g.: files, paths, routes).",
      "name": "truck",
      "semver": null
    },
    {
      "emoji": "📄",
      "entity": "&#1F4C4;",
      "code": ":page_facing_up:",
      "description": "Add or update license.",
      "name": "page-facing-up",
      "semver": null
    },
    {
      "emoji": "💥",
      "entity": "&#x1f4a5;",
      "code": ":boom:",
      "description": "Introduce breaking changes.",
      "name": "boom",
      "semver": "major"
    },
    {
      "emoji": "🍱",
      "entity": "&#1F371",
      "code": ":bento:",
      "description": "Add or update assets.",
      "name": "bento",
      "semver": "patch"
    },
    {
      "emoji": "♿️",
      "entity": "&#9855;",
      "code": ":wheelchair:",
      "description": "Improve accessibility.",
      "name": "wheelchair",
      "semver": "patch"
    },
    {
      "emoji": "💡",
      "entity": "&#128161;",
      "code": ":bulb:",
      "description": "Add or update comments in source code.",
      "name": "bulb",
      "semver": null
    },
    {
      "emoji": "🍻",
      "entity": "&#x1f37b;",
      "code": ":beers:",
      "description": "Write code drunkenly.",
      "name": "beers",
      "semver": null
    },
    {
      "emoji": "💬",
      "entity": "&#128172;",
      "code": ":speech_balloon:",
      "description": "Add or update text and literals.",
      "name": "speech-balloon",
      "semver": "patch"
    },
    {
      "emoji": "🗃️",
      "entity": "&#128451;",
      "code": ":card_file_box:",
      "description": "Perform database related changes.",
      "name": "card-file-box",
      "semver": "patch"
    },
    {
      "emoji": "🔊",
      "entity": "&#128266;",
      "code": ":loud_sound:",
      "description": "Add or update logs.",
      "name": "loud-sound",
      "semver": null
    },
    {
      "emoji": "🔇",
      "entity": "&#128263;",
      "code": ":mute:",
      "description": "Remove logs.",
      "name": "mute",
      "semver": null
    },
    {
      "emoji": "👥",
      "entity": "&#128101;",
      "code": ":busts_in_silhouette:",
      "description": "Add or update contributor(s).",
      "name": "busts-in-silhouette",
      "semver": null
    },
    {
      "emoji": "🚸",
      "entity": "&#128696;",
      "code": ":children_crossing:",
      "description": "Improve user experience / usability.",
      "name": "children-crossing",
      "semver": "patch"
    },
    {
      "emoji": "🏗️",
      "entity": "&#1f3d7;",
      "code": ":building_construction:",
      "description": "Make architectural changes.",
      "name": "building-construction",
      "semver": null
    },
    {
      "emoji": "📱",
      "entity": "&#128241;",
      "code": ":iphone:",
      "description": "Work on responsive design.",
      "name": "iphone",
      "semver": "patch"
    },
    {
      "emoji": "🤡",
      "entity": "&#129313;",
      "code": ":clown_face:",
      "description": "Mock things.",
      "name": "clown-face",
      "semver": null
    },
    {
      "emoji": "🥚",
      "entity": "&#129370;",
      "code": ":egg:",
      "description": "Add or update an easter egg.",
      "name": "egg",
      "semver": "patch"
    },
    {
      "emoji": "🙈",
      "entity": "&#8bdfe7;",
      "code": ":see_no_evil:",
      "description": "Add or update a .gitignore file.",
      "name": "see-no-evil",
      "semver": null
    },
    {
      "emoji": "📸",
      "entity": "&#128248;",
      "code": ":camera_flash:",
      "description": "Add or update snapshots.",
      "name": "camera-flash",
      "semver": null
    },
    {
      "emoji": "⚗️",
      "entity": "&#x2697;",
      "code": ":alembic:",
      "description": "Perform experiments.",
      "name": "alembic",
      "semver": "patch"
    },
    {
      "emoji": "🔍️",
      "entity": "&#128269;",
      "code": ":mag:",
      "description": "Improve SEO.",
      "name": "mag",
      "semver": "patch"
    },
    {
      "emoji": "🏷️",
      "entity": "&#127991;",
      "code": ":label:",
      "description": "Add or update types.",
      "name": "label",
      "semver": "patch"
    },
    {
      "emoji": "🌱",
      "entity": "&#127793;",
      "code": ":seedling:",
      "description": "Add or update seed files.",
      "name": "seedling",
      "semver": null
    },
    {
      "emoji": "🚩",
      "entity": "&#x1F6A9;",
      "code": ":triangular_flag_on_post:",
      "description": "Add, update, or remove feature flags.",
      "name": "triangular-flag-on-post",
      "semver": "patch"
    },
    {
      "emoji": "🥅",
      "entity": "&#x1F945;",
      "code": ":goal_net:",
      "description": "Catch errors.",
      "name": "goal-net",
      "semver": "patch"
    },
    {
      "emoji": "💫",
      "entity": "&#x1f4ab;",
      "code": ":dizzy:",
      "description": "Add or update animations and transitions.",
      "name": "dizzy",
      "semver": "patch"
    },
    {
      "emoji": "🗑️",
      "entity": "&#x1F5D1;",
      "code": ":wastebasket:",
      "description": "Deprecate code that needs to be cleaned up.",
      "name": "wastebasket",
      "semver": "patch"
    },
    {
      "emoji": "🛂",
      "entity": "&#x1F6C2;",
      "code": ":passport_control:",
      "description": "Work on code related to authorization, roles and permissions.",
      "name": "passport-control",
      "semver": "patch"
    },
    {
      "emoji": "🩹",
      "entity": "&#x1FA79;",
      "code": ":adhesive_bandage:",
      "description": "Simple fix for a non-critical issue.",
      "name": "adhesive-bandage",
      "semver": "patch"
    },
    {
      "emoji": "🧐",
      "entity": "&#x1F9D0;",
      "code": ":monocle_face:",
      "description": "Data exploration/inspection.",
      "name": "monocle-face",
      "semver": null
    },
    {
      "emoji": "⚰️",
      "entity": "&#x26B0;",
      "code": ":coffin:",
      "description": "Remove dead code.",
      "name": "coffin",
      "semver": null
    },
    {
      "emoji": "🧪",
      "entity": "&#x1F9EA;",
      "code": ":test_tube:",
      "description": "Add a failing test.",
      "name": "test-tube",
      "semver": null
    },
    {
      "emoji": "👔",
      "entity": "&#128084;",
      "code": ":necktie:",
      "description": "Add or update business logic.",
      "name": "necktie",
      "semver": "patch"
    },
    {
      "emoji": "🩺",
      "entity": "&#x1FA7A;",
      "code": ":stethoscope:",
      "description": "Add or update healthcheck.",
      "name": "stethoscope",
      "semver": null
    },
    {
      "emoji": "🧱",
      "entity": "&#x1f9f1;",
      "code": ":bricks:",
      "description": "Infrastructure related changes.",
      "name": "bricks",
      "semver": null
    },
    {
      "emoji": "🧑‍💻",
      "entity": "&#129489;&#8205;&#128187;",
      "code": ":technologist:",
      "description": "Improve developer experience.",
      "name": "technologist",
      "semver": null
    },
    {
      "emoji": "💸",
      "entity": "&#x1F4B8;",
      "code": ":money_with_wings:",
      "description": "Add sponsorships or money related infrastructure.",
      "name": "money-with-wings",
      "semver": null
    },
    {
      "emoji": "🧵",
      "entity": "&#x1F9F5;",
      "code": ":thread:",
      "description": "Add or update code related to multithreading or concurrency.",
      "name": "thread",
      "semver": null
    },
    {
      "emoji": "🦺",
      "entity": "&#x1F9BA;",
      "code": ":safety_vest:",
      "description": "Add or update code related to validation.",
      "name": "safety-vest",
      "semver": null
    },
    {
      "emoji": "✈️",
      "entity": "&#x2708;",
      "code": ":airplane:",
      "description": "Improve offline support.",
      "name": "airplane",
      "semver": null
    },
    {
      "emoji": "🦖",
      "entity": "&#x2708;",
      "code": ":t-rex:",
      "description": "Code that adds backwards compatibility.",
      "name": "t-rex",
      "semver": null
    }
  ]
}

more info on https://gitmoji.dev
