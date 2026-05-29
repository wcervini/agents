---
name: astro-traduccion
description: Traducción al español de la documentación de Astro. Usar cuando se necesite traducir documentación, guías, tutoriales o contenido de Astro Docs al español siguiendo el glosario y convenciones oficiales del proyecto de traducción.
risk: low
source: community
date_added: "2026-05-29"
---

# Astro Traducción

> Skill para traducir la documentación de Astro al español, siguiendo el glosario, las convenciones de estilo y las buenas prácticas del proyecto oficial de traducción.

## Cuándo usar esta skill

- Traducir páginas de la documentación de Astro (`astro.build/docs`) del inglés al español
- Revisar traducciones existentes para mantener consistencia con el glosario
- Traducir contenido de Starlight u otros proyectos del ecosistema Astro
- Estandarizar términos técnicos siguiendo el glosario acordado por la comunidad de traducción

## Qué hace esta skill

1. **Aplica el glosario oficial** — usa las traducciones acordadas para conceptos de Astro y términos técnicos
2. **Respeta las convenciones de estilo** — formato de títulos, comillas, enlaces y ejemplos
3. **Mantiene consistencia** — asegura que términos como `islands`, `frontmatter`, `slots` se traduzcan uniformemente
4. **Adapta el contenido** — traduce comentarios en ejemplos, adapta rutas y variables cuando sea necesario

## Glosario

### Palabras que NO se traducen

| Término | Condición |
|---------|-----------|
| `Fragment` | Cuando se refiere a `<Fragment> </Fragment>` o `<> </>` |
| `frontmatter` | Siempre |
| `props` | Cuando se refiere a propiedades/atributos de un componente |
| `slot` | Cuando se refiere a la etiqueta `<slot/>` |
| `framework` | Siempre (no traducir como "marco de trabajo") |
| `front-end` / `back-end` | Siempre |
| `middleware` | Siempre |
| `hook` | Siempre |
| `headless` | Siempre |
| `glob` / `globbing` | Siempre |

### Traducciones de conceptos de Astro

| Inglés | Español |
|--------|---------|
| Astro Islands | Islands de Astro |
| Content Collections | Colecciones de contenido |
| Content Layer | Capa de contenido |
| Fonts | Fuentes o Tipografías |
| Island architecture | Arquitectura de islands |
| Sessions | Sesiones |

### Traducciones comunes

| Inglés | Español |
|--------|---------|
| assets | recursos |
| breaking changes | cambios que rompen compatibilidad / no retrocompatibles |
| build | creación / construcción / compilación |
| bundle / to bundle | agrupar |
| changelog | registro de cambios |
| CLI / command line interface | CLI / interfaz de línea de comandos |
| client-side | lado del cliente |
| code fences | delimitador de código |
| deprecation / to deprecate | descatalogado / depreciar |
| docs | documentación |
| endpoint | punto de conexión |
| export / to export | exportación / exportar |
| feedback | comentarios |
| flag | opción |
| footer | pie de página |
| header | cabecera |
| heading | título |
| import / to import | importación / importar |
| issue | problema / issue (GitHub) |
| layout | diseño / estructura |
| on-demand rendering | renderizado bajo demanda |
| overlay | superposición |
| package | paquete |
| pattern | patrón |
| placeholder | marcador de posición |
| plugin | plugin / extensión |
| preset | configuración predefinida |
| prop | propiedad |
| output | salida |
| re-render | volver a renderizar |
| recipes | recetas |
| renderer | motor de renderizado |
| rendering | renderizado |
| repository | repositorio |
| routing | enrutamiento |
| runtime | motor de ejecución / entorno de ejecución |
| scope | alcance |
| scoped | con alcance limitado |
| server-side | lado del servidor |
| template | plantilla |
| to build | crear / construir / compilar |
| to commit | confirmar |
| to fetch | obtener |
| to prefetch | precargar |
| to prerender | pre-renderizar |
| to render | renderizar |
| to slot | insertar |
| to store | almacenar |
| to style | aplicar estilos |
| to support | admitir / ser compatible con |
| to type | escribir / escribir tipo |
| to wrap | envolver |
| type safe | seguridad de tipos |
| UI | IU |
| update | actualización |
| upgrade | mejora |
| view transitions | transiciones de vista |

## Convenciones de estilo

### Títulos
- Usar mayúscula solo al inicio del título (no capitalizar cada palabra como en inglés)
- Preferir infinitivo sobre imperativo

### Comillas
- Usar comillas tipográficas españolas: `« texto »`
- Unicode: `«` = U+00ab, `»` = U+00bb

### Código como palabra
- Reemplazar `` `código` `` por una palabra en español y añadir la versión en código entre paréntesis
- Ejemplo: `` `path` `` → `path del archivo (`path`)`

### Enlaces internos
- Reemplazar `/en/` por `/es/` en todas las URLs
- Traducir el fragmento después de `#` (corresponde al id del título)

### Enlaces externos
- Traducir siempre el texto/ancla del enlace
- Mantener el enlace original salvo que exista versión en español
- Si no hay versión en español, añadir `(Inglés)` después del enlace

### Ejemplos
- Traducir siempre los comentarios en el código
- Adaptar rutas y nombres de variables si facilita la comprensión
- Actualizar el resaltado de código si es necesario

## Recursos

- [Discord oficial de Astro](https://astro.build/chat) — hilo `#i18n-es`
- [Fundeu](https://www.fundeu.es/) — consultar términos en español
- [UVL - Universitat de València](https://www.uv.es/pls/uvi/) — terminología española
- Consultar MDN, React, Vue docs en español como referencia
