---
description: Revisa cambios staged y gestiona commit, versión y release con Conventional Commits
---

Carga la skill **cc** y ejecuta el siguiente flujo interactivo:

## Paso 1: Analizar cambios

Ejecuta `git diff --staged` y analiza los archivos cambiados. Identifica tipo de cambio, scope, breaking changes y issues relacionados.

## Paso 2: Preguntar al usuario

Pregunta qué quiere hacer:

1. **solo mensaje** — generar solo el mensaje de commit sin ejecutar nada más
2. **hacer commit** — generar mensaje y ejecutar el commit
3. **actualizar versión** — generar mensaje, commit, y actualizar versión

## Paso 3: Según la respuesta

### Si es "solo mensaje":
Muestra el mensaje de commit propuesto y termina.

### Si es "hacer commit":
1. Muestra el mensaje propuesto
2. Pregunta confirmación antes de ejecutar
3. Ejecuta `git commit` con el mensaje

### Si es "actualizar versión":
1. Muestra el mensaje propuesto
2. Pregunta confirmación
3. Ejecuta el commit
4. Pregunta si quiere actualizar el **CHANGELOG.md**
5. Pregunta si quiere actualizar el **README.md**
6. Pregunta si quiere hacer **push** (incluyendo tag si existe)
7. Pregunta si quiere publicar una **release** en GitHub

## Formato del mensaje

Usa **Conventional Commits** con **gitmoji** al inicio:

```
✨ feat(scope): descripción

- bullet points con cambios

Closes #123
```

Pregunta una cosa a la vez y espera la respuesta antes de continuar.