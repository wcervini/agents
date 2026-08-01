---
description: Crea una rama, hace commits semánticos y genera una Pull Request para que el usuario revise y apruebe. NUNCA push directo a main.
---

NUNCA hagas commits ni push directo a `main`. Todo cambio debe pasar por Pull Request.
El usuario revisa, prueba y aprueba el merge. No hacer bypass de protecciones de GitHub Actions.

$ARGUMENTS

## Workflow

1. Si no estás en `main`, haz checkout: `git checkout main && git pull origin main`
2. Crea rama nueva: `git checkout -b <tipo>/<descripcion-breve>` (ej: `fix/car-translations`, `feat/nueva-funcionalidad`)
3. Inspecciona estado completo: `git status --short`, `git diff --stat`, `git diff`, `git log --oneline -10`
4. Identifica grupos de archivos relacionados por intención (feature, fix, refactor, docs, chore)
5. Para cada grupo: `git add <files>` y commit con identidad explícita:
   `git -c user.name="opencode-ai" -c user.email="wcervini+opencode-ai@gmail.com" -c user.signingkey="$HOME/.ssh/opencode_signing.pub" commit -S -m "<gitmoji> <tipo>(<alcance>): <descripcion>"`
6. Una vez todos los commits listos: `git push origin <nombre-rama>`
7. Crea PR: `gh pr create --base main --head <rama> --title "<titulo>" --body "<descripcion de los cambios>"`
8. Devuelve al usuario el enlace de la PR
9. Si la PR esta cerrada verificar si se hizo el merge a main y eliminar la rama.
10. Si no se hizo Merge a Main preguntar al usuario si puedes eliminar la rama donde se hicieron los cambios

## Reglas

- NUNCA hacer commits ni push directo a `main`
- No hacer bypass de protecciones de GitHub Actions
- No usar `no-verify`
- No hacer force push
- Antes de commit, revisar archivos sensibles (`.env`, tokens, credenciales). Si aparece alguno, parar y preguntar
- No revertir cambios existentes
- register por el comando /super-commit

