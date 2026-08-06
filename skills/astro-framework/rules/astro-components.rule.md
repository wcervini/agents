---
description: Rules for writing Astro components
globs:
  - "**/*.astro"
---

# Astro Component Rules

## Component Structure

Always use the frontmatter pattern with proper separation:

```astro
---
// 1. Imports
import Layout from '../layouts/Layout.astro';
import { getCollection } from 'astro:content';

// 2. Props interface
interface Props {
  title: string;
  description?: string;
}

// 3. Destructure props with defaults
const { title, description = 'Default' } = Astro.props;

// 4. Data fetching and logic
const posts = await getCollection('blog');
---

<!-- 5. Template -->
<Layout title={title}>
  <h1>{title}</h1>
</Layout>

<!-- 6. Scoped styles -->
<style>
  h1 { color: navy; }
</style>
```

## Rust Compiler (Astro 7)

Astro 7 compiles `.astro` files with a Rust-based compiler that enforces **strict HTML**. Unlike the legacy compiler, it does **not** autocorrect malformed markup:

- **All tags must be properly closed** — void elements (`<img>`, `<input>`, `<br>`, `<meta>`, `<link>`, etc.) are the only exception
- **No auto-closing or tag fixing** — Astro will not silently repair unclosed or mismatched tags; it errors instead
- **Strict nesting** — mismatched or misnested tags produce compile errors rather than being tolerated

```astro
<!-- MUST: close every non-void tag -->
<div>
  <p>Content</p>
</div>

<!-- MUST: void elements stay self-closing -->
<img src={hero} alt="Hero" />
<input type="text" name="email" />
```

```astro
<!-- NEVER: unclosed or mismatched tags -->
<div>
  <p>Content
</div>
```

## MUST DO

- Define `interface Props` for type safety
- Use destructuring with defaults for optional props
- Keep frontmatter logic minimal and focused
- Use slots for composable content
- Use `class:list` for conditional classes
- Import components and assets at the top

## MUST NOT DO

- Access browser APIs (window, document) in frontmatter - runs on server
- Use side effects in frontmatter - runs on every render
- Mix UI framework components without client directives
- Forget alt text on images
- Use inline styles when scoped styles work
- Skip TypeScript interfaces for Props
- Leave tags unclosed or mismatched — the Rust compiler (Astro 7) does not autocorrect HTML

## Slots Pattern

```astro
---
// Wrapper.astro
interface Props {
  title: string;
}
const { title } = Astro.props;
const hasFooter = Astro.slots.has('footer');
---

<article>
  <header><h1>{title}</h1></header>
  <main><slot /></main>
  {hasFooter && <footer><slot name="footer" /></footer>}
</article>
```

## Dynamic Attributes

```astro
---
const id = "main";
const isActive = true;
---

<div
  id={id}
  class:list={['base', { active: isActive }]}
  data-active={isActive}
>
  Content
</div>
```

## Conditional Rendering

```astro
---
const show = true;
const items = ['a', 'b', 'c'];
---

{show && <p>Visible</p>}
{show ? <p>Yes</p> : <p>No</p>}

<ul>
  {items.map(item => <li>{item}</li>)}
</ul>
```
