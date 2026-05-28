---
name: vscode-snippets
description: Genera snippets de VS Code a partir de código seleccionado por el usuario en el documento activo. Analiza el código, identifica patrones y variables, y produce un snippet JSON listo para usar.
metadata:
  tags: vscode, snippets, code-generation, productivity
  platforms: OpenCode, Claude, Gemini, ChatGPT
---

# VS Code Snippets Generator

## When to use this skill

- Cuando el usuario selecciona código en su editor y pide crear un snippet de VS Code a partir de esa selección.
- Cuando necesitas generar snippets reutilizables con placeholders y tabstops (`$1`, `$2`, `${1:default}`, `${TM_FILENAME}`, etc.).
- Cuando quieres convertir código repetitivo en snippets reutilizables para el equipo.

## Instructions

### Step 1: Obtener el código seleccionado

Pídele al usuario que pegue el código seleccionado o que comparta el contenido del documento activo. El código puede ser cualquier lenguaje: JavaScript, TypeScript, Python, HTML, CSS, React, Vue, etc.

### Step 2: Analizar el código y detectar patrones

Identifica en el código qué partes son:

- **Constantes fijas** — se quedan igual en el snippet.
- **Variables/parámetros** — se convierten en placeholders (`$1`, `$2`, `${3:defaultValue}`).
- **Nombres de funciones/clases/variables** — se convierten en placeholders con nombres descriptivos.
- **Repeticiones** — si un valor aparece varias veces, usa el mismo `$N` para mantener sincronización.
- **Opciones** — si aplican valores alternativos, usa `${1|opcion1,opcion2|}`.
- **Variables de VS Code** — usa `${TM_FILENAME}`, `${TM_SELECTED_TEXT}`, `${CURRENT_YEAR}`, etc. cuando tenga sentido.

### Step 3: Generar el snippet JSON

Genera el snippet en formato JSON de VS Code:

```jsonc
{
  "Nombre del Snippet": {
    "prefix": "trigger-text",
    "body": [
      "línea 1 del snippet",
      "línea 2 con $1 y ${2:default}",
      "  línea con indentación",
      "línea con ${TM_FILENAME_BASE}",
    ],
    "description": "Descripción clara de lo que hace el snippet",
  },
}
```

**Reglas para el `body`**:

- Cada línea del código debe ser un string independiente en el array `body`.
- Usa `$1`, `$2`, etc. para tabstops en orden lógico (lo primero que se edita es `$1`).
- Usa `${N:default}` cuando tenga sentido proporcionar un valor por defecto.
- Escapa dobles comillas como `\"` dentro de las líneas del body.
- Usa `\t` para tabs dentro del snippet si es necesario.
- Si el código usa template literals con `${}`, escríbelos como `\${}` para evitar que VS Code los interprete como variables de snippet.

### Step 4: Elegir el scope del snippet

Pregunta o sugiere el scope del lenguaje:

- `javascript`, `typescript`, `javascriptreact`, `typescriptreact`
- `python`, `html`, `css`, `scss`, `json`
- `vue`, `svelte`, `astro`
- O simplemente omite `scope` para que funcione en todos los lenguajes.

### Step 5: Instrucciones de uso

Indica al usuario dónde guardar el snippet:

1. **Global**: `%APPDATA%\Code\User\snippets\{language}.json`
2. **Proyecto**: `.vscode\{language}.code-snippets`
3. O usando la paleta de comandos: `Preferences: Configure User Snippets`

### Step 6: Entrega final

Proporciona:

1. El JSON completo del snippet listo para copiar y pegar.
2. El nombre del snippet, prefix y descripción sugeridos.
3. Una demostración de cómo se expande el snippet (el código resultante tras rellenar los placeholders).

## Examples

### Ejemplo 1: React Functional Component

**Código seleccionado**:

```tsx
const Button = ({ label, onClick, variant = "primary" }) => {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
};

export default Button;
```

**Snippet generado**:

```json
{
  "React Functional Component": {
    "prefix": "rfc",
    "body": [
      "const ${1:ComponentName} = ({ ${2:props} }) => {",
      "  return (",
      "    <${3:div}>",
      "      ${4:content}",
      "    </${3:div}>",
      "  );",
      "};",
      "",
      "export default ${1:ComponentName};"
    ],
    "description": "React Functional Component con export default"
  }
}
```

**Expanded (después de completar los placeholders)**:

```tsx
const MyButton = ({ label, onClick }) => {
  return (
    <button className="btn btn-default">
      Click me
    </button>
  );
};

export default MyButton;
```

> Tab order: `$1` (ComponentName) → `$2` (props) → `$3` (tag) → `$4` (content)

### Ejemplo 2: Función async con try/catch

**Código seleccionado**:

```typescript
async function fetchUserData(userId: string): Promise<UserData> {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) throw new Error("Network error");
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch user data:", error);
    throw error;
  }
}
```

**Snippet generado**:

```json
{
  "Async Function with Try Catch": {
    "prefix": "async-fn",
    "body": [
      "async function ${1:functionName}(${2:params}): ${3:Promise<ReturnType>} {",
      "  try {",
      "    const response = await fetch(`${4:url}`);",
      "    if (!response.ok) throw new Error('${5:Error message}');",
      "    return await response.json();",
      "  } catch (error) {",
      "    console.error('${6:Failed to fetch}:', error);",
      "    throw error;",
      "  }",
      "}"
    ],
    "description": "Función async con try/catch y fetch"
  }
}
```

**Expanded (después de completar los placeholders)**:

```typescript
async function getUser(id: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error("User not found");
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch:", error);
    throw error;
  }
}
```

> Tab order: `$1` (functionName) → `$2` (params) → `$3` (return type) → `$4` (url) → `$5` (error message) → `$6` (log message)

### Ejemplo 3: Choice Dropdown

Usa `${N|opt1,opt2,opt3|}` para crear un dropdown.

**Snippet**:

```json
{
  "React useState": {
    "prefix": "usestate",
    "body": [
      "const [${1:state}, set${1:state}] = useState${2:>(null)};",
      "",
      "${3:// TODO: add ${4:handler}}"
    ],
    "description": "React useState hook con setter"
  }
}
```

**Expanded (con opciones elegidas)**:

```typescript
const [count, setCount] = useState<number>();

// TODO: add incrementHandler
```

### Ejemplo 4: Transformaciones

Usa regex para transformar el valor de un placeholder.

**Snippet**:

```json
{
  "Console Method": {
    "prefix": "clog",
    "body": [
      "console.${1|log,warn,error,info|}(${2:message});",
      "// File: ${TM_FILENAME_BASE} | Line: ${TM_LINE_NUMBER}"
    ],
    "description": "Console log con info de contexto"
  }
}
```

**Expanded**:

```typescript
console.warn('Invalid input');
// File: validation.ts | Line: 42
```

---

## VS Code Variables

Variables disponibles en todos los snippets:

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `${TM_FILENAME}` | Nombre completo del archivo | `index.ts` |
| `${TM_FILENAME_BASE}` | Nombre sin extensión | `index` |
| `${TM_DIRECTORY}` | Directorio del archivo | `/src/components` |
| `${TM_FILEPATH}` | Ruta completa del archivo | `/project/src/index.ts` |
| `${TM_LINE_NUMBER}` | Número de línea actual | `42` |
| `${TM_CURRENT_LINE}` | Contenido de la línea actual | `const x = 1;` |
| `${TM_SELECTED_TEXT}` | Texto seleccionado | `hola mundo` |
| `${TM_CURRENT_WORD}` | Palabra bajo el cursor | `funcion` |
| `${CURRENT_YEAR}` | Año actual | `2026` |
| `${CURRENT_MONTH}` | Mes actual (00-12) | `05` |
| `${CURRENT_DATE}` | Fecha actual (DD/MM/YYYY) | `28/05/2026` |
| `${CURRENT_DAY_NAME}` | Nombre del día | `Thursday` |
| `${CLIPBOARD}` | Contenido del portapapeles | `texto copiado` |
| `${BLOCK_COMMENT_START}` |Inicio comentario bloque | `/*` |
| `${BLOCK_COMMENT_END}` | Fin comentario bloque | `*/` |
| `${LINE_COMMENT}` | Comentario de línea | `//` |

**Escapar variables**: Usa `\${VAR}` para mostrar el símbolo `$` literal.

---

## Choosing the Prefix

El prefix es el texto que disparará el snippet. Debe ser:

### Criterios para un buen prefix

| Criterio | Ejemplo bueno | Ejemplo malo |
|----------|---------------|--------------|
| **Corto** (2-6 chars) | `rfc`, `afn`, `try` | `react-functional-component` |
| **Memorable** | `pfx` → `prefix` | `pref` (ambigio) |
| ** Único** | `usestate` | `state` (demasiado genérico) |
| **Consistente** | `fn`, `fnc`, `func` | mezlcar naming |

### Convenciones comunes

```
rfce      → React Functional Component Export
afn       → Async Function
trycatch  → Try Catch Block
imr       → Import React
clog      → Console Log
prop      → Prop types
hook      → Custom hook (useXXX)
css       → CSS class
req       → HTTP Request
```

### Scope por prefix

```
snake_case  → python, go
kebab-case  → css, html
camelCase   → javascript, typescript
PascalCase  → React components, classes
```

---

## Scope: Global vs Language-Specific

### Snippet global

```json
{
  "Global Snippet": {
    "prefix": "global",
    "body": ["..."],
    "description": "Works in all files"
  }
}
```

### Snippet por lenguaje

```json
{
  "Python Snippet": {
    "prefix": "def",
    "body": ["def ${1:name}(${2:args}):", "    ${3:pass}"],
    "description": "Python function definition",
    "scope": "python"
  }
}
```

### Lenguajes comunes

| Language | Scope identifier |
|----------|-----------------|
| JavaScript | `javascript` |
| TypeScript | `typescript` |
| React JSX | `javascriptreact` |
| React TSX | `typescriptreact` |
| Python | `python` |
| HTML | `html` |
| CSS | `css` |
| JSON | `json` |
| Markdown | `markdown` |
| Vue | `vue` |
| Svelte | `svelte` |
| Astro | `astro` |

### Dónde guardar snippets

| Tipo | Ubicación | Uso |
|------|------------|-----|
| **Global** | `%APPDATA%\Code\User\snippets\{lang}.json` | Personal, todos los proyectos |
| **Proyecto** | `.vscode\{lang}.code-snippets` | Compartido con equipo |
| **Extensión** | Extensión marketplace | Compartir públicamente |

### Recomendación

> Para snippets de equipo: crear en `.vscode/snippets.code-snippets` (global para el proyecto) o crear una extensión.

---

## Common Mistakes

### Evita estos errores

❌ **Usar `$0` incorrectamente**

```json
// ❌ Mal - $0 al final no es necesario
"body": ["const ${1:name} = ${2:value};", "$0"]

// ✅ Bien - $0 es para la posición final del cursor
"body": ["const ${1:name} = ${2:value};"]
```

❌ **No sincronizar variables relacionadas**

```json
// ❌ Mal - cada $1 es independiente
"body": ["const ${1:first} = ${1:second};"]

// ✅ Bien - mismo placeholder = mismo valor
"body": ["const ${1:name} = ${1:name};"]
```

❌ **Olvidar escapar `$` en template literals**

```json
// ❌ Mal - ` ${value}` se interpreta como variable
"body": ["const \${name} = `\${value}`;"]

// ✅ Bien - se muestra como: const ${name} = `${value}`;
```

❌ **Números de tabstop no secuenciales**

```json
// ❌ Mal - salta números
"body": ["$1", "$3", "$5"]

// ✅ Bien - secuencial o al menos lógico
"body": ["$1", "$2", "$3"]
```

❌ **Prefijo demasiado largo**

```json
// ❌ Mal
"prefix": "create-react-functional-component-with-props"

// ✅ Bien
"prefix": "rfcp"
```

❌ **Olvidar el scope para snippets específicos**

```json
// ❌ Mal - snippet de Python sin scope
"prefix": "def"

// ✅ Bien
"prefix": "def",
"scope": "python"
```

❌ **JSON mal formado**

```json
// ❌ Mal - comas faltantes
{
  "Snippet": {
    "prefix": "test"
    "body": ["..."]
  }
}

// ✅ Bien
{
  "Snippet": {
    "prefix": "test",
    "body": ["..."]
  }
}
```

### Checklist antes de entregar

- [ ] ¿El JSON es válido? (usa jsonlint si tienes duda)
- [ ] ¿Los tabstops `$1`, `$2`, etc. están en orden lógico?
- [ ] ¿Las comillas dentro del body están escapadas con `\`?
- [ ] ¿El prefix es corto y memorable?
- [ ] ¿La descripción explica qué hace el snippet?
- [ ] ¿El scope es correcto si es lenguaje-específico?

---

## Tips

- **Prioriza los tabstops en orden lógico** de edición: `$1` es lo primero que el usuario editará.
- **Usa valores por defecto** en `${N:default}` que sean representativos del tipo de dato esperado.
- **Mantén la indentación original** del código seleccionado.
- **Si el código es muy grande (>30 líneas)**, sugiere dividirlo en múltiples snippets más pequeños y enfocados.
- **Para snippets con múltiples cursores**, indica al usuario que puede usar `$1`, `$2` múltiples veces para que el mismo valor se edite sincronizadamente.
- **Para snippets condicionales**, usa choice placeholders `${1|a,b,c|}` en vez de múltiples snippets similares.
- **Usa transformaciones** para convertir entre camelCase, PascalCase, snake_case automáticamente.
