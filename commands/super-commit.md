---
description: Eres un especialista en repositorio GIT
---


$ARGUMENTS

## Workflow

1. Inspecciona estado completo: `git status --short`, `git diff --stat`, `git diff`, `git log --oneline -10`
2. Identifica grupos de archivos relacionados por intención (feature, fix, refactor, docs, chore)
3. Para cada grupo: `git add <files>` y commit con identidad explícita:
   `git -c user.name="opencode-ai" -c user.email="wcervini+opencode-ai@gmail.com" -c user.signingkey="$HOME/.ssh/opencode_signing.pub" commit -S -m "<gitmoji> <tipo>(<alcance>): <descripcion>"`
6. Una vez todos los commits listos: `git push origin <nombre-rama>`
7. Crea PR si no puedes hacer Push en Main
8. Devuelve al usuario el enlace de la PR
9. Si la PR está cerrada, verifica si se hizo el merge a main y elimina la rama.
10. Si no se hizo merge a main, pregunta al usuario si puedes eliminar la rama donde se hicieron los cambios.

## Reglas

- No hacer bypass de protecciones de GitHub Actions
- No usar `no-verify`
- No hacer force push
- Antes de commit, revisar archivos sensibles (`.env`, tokens, credenciales). Si aparece alguno, parar y preguntar
- No revertir cambios existentes
- Para la composicion del mensaje de cada commit, solicita apoyo a la skill `msg-commit` (agrupacion por intencion y commits semanticos con gitmoji)
