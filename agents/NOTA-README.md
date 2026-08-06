# 📌 NOTA — Sub-agentes vs skills

## ¿Esta carpeta la carga opencode?

No directamente como sub-agentes. opencode **no** lee agentes desde
`~/.agents/agents/`; lee los sub-agentes globales desde
`~/.config/opencode/agents/<nombre>.md`.

## Dónde carga cada tipo

| Tipo | Carpeta que opencode SÍ lee |
|------|------------------------------|
| **Skills** (external, auto-loaded) | `~/.agents/skills/<nombre>/SKILL.md` |
| **Sub-agentes** (global) | `~/.config/opencode/agents/<nombre>.md` |
| **Skills** configuradas | `opencode.json → skills.paths` |

## Cómo activar un sub-agente de aquí

Copiarlo al directorio real que opencode lee y reiniciar:

    cp ~/.agents/agents/<nombre>.md ~/.config/opencode/agents/
    # reinicia opencode (la config no se recarga en caliente)

## Cómo activar una skill de aquí (auto-load sin config)

Colocarla en `~/.agents/skills/<nombre>/SKILL.md` (ya es la ubicación
que opencode escanea por defecto).
