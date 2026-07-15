---
name: commits
description: Generates commit messages using MCP auto-commit with gitmoji and Conventional Commits.
metadata:
  tags: git, conventional-commits, gitmoji, mcp, frontend-architect, version-control
  platforms: OpenCode, Claude, Gemini, ChatGPT, VSCode
---

# Conventional Commits Architect with Gitmoji

## When to use this skill

- **Before committing**: When you need to generate an automated commit message.
- **Git workflow standardization**: To ensure consistent commits with gitmoji and Conventional Commits.
- **MCP integration**: Uses the auto-commit MCP to generate messages with AI.

## MCP Integration

This skill uses the MCP server `auto-commit` which:
1. Analyzes changes with `git diff`
2. Generates messages with gitmoji using OpenCode Zen
3. Executes `git commit` automatically

**Available MCP tool:** `auto-commit_git-changes-commit-message`
- Parameter: `autoCommitPath` (optional) - path to analyze
- If not provided, uses the current directory

## Specification

### Required Structure

```
<gitmoji> <type>[optional scope]: <description>

[optional body]

[optional footer]
```

### Commit Types

| Type       | Description                                                               |
| ---------- | ------------------------------------------------------------------------- |
| `feat`     | A new feature for the user                                                |
| `fix`      | A bug fix that affects the user                                           |
| `docs`     | Documentation only changes                                                |
| `style`    | Changes that do not affect code meaning (formatting, whitespace, etc.)    |
| `refactor` | Code refactoring that neither fixes a bug nor adds a feature              |
| `perf`     | Changes that improve performance                                          |
| `test`     | Adding or correcting tests                                                |
| `build`    | Changes affecting the build system or dependencies                        |
| `ci`       | Changes to CI configuration files and scripts                             |
| `chore`    | Other changes that do not modify src or test files                        |

### Description Rules

1. **Gitmoji required** at the beginning of the first line (e.g.: `✨`, `🐛`, `📝`)
2. **Maximum 50 characters** recommended for the first line (hard limit: 72, both excluding the gitmoji)
3. **No period at the end**
4. **Use imperative mood**: "add feature" not "added feature" / "adds feature"
5. **Do not capitalize** the first letter
6. **No semicolon** at the end

### Scope

The scope is optional and identifies the section of the affected codebase:

```
feat(auth): add OAuth2 login
fix(api): resolve race condition in user endpoint
docs(readme): update installation instructions
```

**Common scopes**:

- `api`, `ui`, `db`, `auth`, `config`, `build`, `deps`, `docs`, `tests`
- Project-specific module names

### Body

- Separate with a blank line after the description
- Maximum 72 characters per line
- Explain **what** and **why**, not **how**
- Use bullet points with `-` for multiple lines

### Footer

- Separate with a blank line after the body
- Reference issues and PRs:
  - `Closes #123`
  - `Fixes #456`
  - `Refs #789`
- List breaking changes:
  - `BREAKING CHANGE: remove deprecated API`

### Breaking Changes

**Method 1** (in footer):

```
BREAKING CHANGE: the authentication API now requires a JWT token.
Migration: update calls to /api/v2/auth.
```

**Method 2** (in type/scope):

```
feat!: remove support for Node 16
```

---

## Instructions

### Step 1: Generate commit with MCP

Run the MCP tool `auto-commit_git-changes-commit-message`:

```
Usage: auto-commit_git-changes-commit-message
Parameter autoCommitPath: (optional) specific path or empty for current directory
```

The MCP will return:
- List of modified, added, and deleted files
- A commit message with gitmoji in conventional commits format
- The commit is executed automatically

### Step 2: Verify the result

The generated message must comply with:
- ✅ Gitmoji emoji at the start (✨, 🐛, 📝, ♻️, 🔧, etc.)
- ✅ Correct type (feat, fix, docs, refactor, etc.)
- ✅ Maximum 50 characters recommended (hard limit: 72) in the first line
- ✅ No period at the end
- ✅ Imperative mood ("add" not "added")

## Examples

### Example 1: New feature

**Diff**: Adds `/api/users` endpoint with pagination.

```
✨ feat(api): add paginated users endpoint

- GET /api/users?page=1&limit=20
- Returns { data, pagination }
- Validates query parameters
- Includes rate limiting

Closes #234
```

### Example 2: Bug fix

**Diff**: Fixes memory leak in the job worker.

```
🐛 fix(worker): resolve memory leak in job processor

The processor was not releasing event listeners on completion.
Now properly cleans up all listeners after each job.

Fixes #567
```

### Example 3: Refactoring

**Diff**: Extracts authentication logic to a dedicated module.

```
♻️ refactor(auth): extract auth logic to dedicated module

- Move token validation from controller to AuthService
- Add refresh token rotation
- Reduce cognitive complexity by 40%
```

### Example 4: Breaking change

**Diff**: Removes legacy v1 API.

```
💥 feat!: remove deprecated v1 API endpoints

BREAKING CHANGE: All /api/v1/* endpoints removed.
Migration guide: see docs/migration-v2.md

Closes #890
```

### Example 5: Documentation only

**Diff**: Updates README with new installation instructions.

```
📝 docs(readme): update installation instructions

- Add Docker setup steps
- Include environment variables reference
- Fix outdated npm commands
```

### Example 6: Dependency changes

**Diff**: Upgrades Express from 4.x to 5.x.

```
⬆️ build(deps): upgrade Express to v5

- Migrate to async handlers
- Update middleware signatures
- Required for Node 20+ compatibility
```

### Example 7: CI fix

**Diff**: Fixes GitHub Actions pipeline.

```
👷 ci: fix test runner timeout

- Increase timeout from 2min to 5min for integration tests
- Add retry logic for flaky tests
- Update Node version matrix
```

### Example 8: General maintenance

**Diff**: Cleans up old console.log statements and unused imports.

```
🧹 chore: remove debug logs and unused imports

- Remove console.log statements from payment module
- Clean up unused lodash imports
- Sort Tailwind CSS class order
```

### Example 9: Code style

**Diff**: Formats codebase with Prettier and reorganizes imports.

```
🎨 style: format codebase with Prettier

- Apply consistent indentation across all files
- Reorder imports alphabetically
- Add trailing commas to function params
```

### Example 10: Revert changes

**Diff**: Reverts a faulty deployment that caused production issues.

```
⏪️ revert: rollback user profile feature

The deployment caused database connection timeouts
under load. Reverting to the previous stable version.

Fixes #892
```

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────┐
│  <gitmoji> type(scope): description                     │
│                                                         │
│  [optional body]                                        │
│                                                         │
│  [optional footer: Closes #N, BREAKING CHANGE: ...]     │
└─────────────────────────────────────────────────────────┘

feat     → New feature
fix      → Bug fix
docs     → Documentation
style    → Formatting (not logic)
refactor → Code restructuring
perf     → Performance
test     → Tests
build    → Build/dependencies
ci       → CI/CD
chore    → General maintenance

Atomic commits: one type, one purpose
```

---

## Anti-Patterns (avoid)

❌ `Fixed the bug`
❌ `Updated code`
❌ `WIP`
❌ `asdasdasd`
❌ `feat: Add new feature for the user`
❌ `fix: Fixed critical issue in the system.`

✅ `✨ feat(auth): add JWT refresh token rotation`
✅ `🐛 fix(api): resolve race condition in user endpoint`
✅ `📝 docs: update API documentation`

---

## Related Tools

- **commitlint**: Validates commit messages against Conventional Commits rules. Useful as a git hook to enforce consistency.
- **commitizen**: Interactive CLI that helps craft Conventional Commits with prompts for type, scope, and description.
- **standard-version**: Automates version bumping and changelog generation from Conventional Commits.
- **semantic-release**: Fully automated package publishing — analyzes commits to determine version bumps and generates release notes.
- **husky** + **lint-staged**: Pair used to run commitlint and other validators as git hooks before each commit.

---

## Gitmoji Reference

Commits must start with the corresponding emoji. See the full list in [references/GITMOJI.md](references/GITMOJI.md).

**Most common gitmojis:**

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

More info on [gitmoji.dev](https://gitmoji.dev)
