# Content Collections

Content collections provide type-safe content management with schema validation using Zod.

## Setup (Astro 5+ Content Layer API)

### Directory Structure

```
src/
├── content.config.ts    # Collection schemas (Astro 5+)
├── content/
│   ├── blog/            # Blog collection
│   │   ├── post-1.md
│   │   ├── post-2.mdx
│   │   └── drafts/      # Subdirectories supported
│   │       └── draft-1.md
│   └── authors/         # Another collection
│       └── john.json
```

> **Note:** In Astro 5+, the config file is `src/content.config.ts` (not `src/content/config.ts`). Collections use `loader` instead of `type`.

### Configuration File (Astro 5+ — Recommended)

```typescript
// src/content.config.ts
import { defineCollection } from 'astro:content';
import { glob, file } from 'astro/loaders';
import { z } from 'astro/zod';

const blogCollection = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.{md,mdx}' }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    heroImage: z.string().optional(),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
    author: z.string(),
  }),
});

const authorsCollection = defineCollection({
  loader: file('src/data/authors.json'),
  schema: z.object({
    name: z.string(),
    email: z.string().email(),
    bio: z.string(),
    avatar: z.string().url(),
    social: z.object({
      twitter: z.string().optional(),
      github: z.string().optional(),
    }).optional(),
  }),
});

export const collections = {
  blog: blogCollection,
  authors: authorsCollection,
};
```

### Legacy Configuration (Astro 4)

```typescript
// src/content/config.ts (legacy path)
import { defineCollection, z } from 'astro:content';

const blogCollection = defineCollection({
  type: 'content', // Markdown/MDX files
  schema: z.object({
    title: z.string(),
    pubDate: z.coerce.date(),
  }),
});

const authorsCollection = defineCollection({
  type: 'data', // JSON/YAML files
  schema: z.object({
    name: z.string(),
    email: z.string().email(),
  }),
});

export const collections = { blog: blogCollection, authors: authorsCollection };
```

## Built-in Loaders (Astro 5+)

### `glob()` — Multiple files

Loads entries from directories of Markdown, MDX, JSON, YAML, or TOML files:

```typescript
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({
    base: './src/content/blog',
    pattern: '**/*.{md,mdx}',
  }),
  schema: z.object({ title: z.string() }),
});
```

Options: `pattern`, `base`, `generateId()` (custom ID generation), `retainBody` (set `false` to exclude raw body), `deferRender` (Astro 7.1+).

#### `deferRender` (Astro 7.1+)

By default, `glob()` renders each Markdown entry eagerly during content sync and stores the HTML in the data store. For very large collections this can exhaust memory. Set `deferRender: true` to defer rendering until each entry is rendered in a page (the same on-demand path MDX uses). This bounds memory during `astro build` at the cost of no longer caching the rendered HTML. Does not apply to MDX, Markdoc, or data entries.

```typescript
const docs = defineCollection({
  loader: glob({
    pattern: '**/*.md',
    base: './src/data/docs',
    deferRender: true,
  }),
});
```

### `file()` — Single file

Loads entries from a single JSON, YAML, or TOML file:

```typescript
import { file } from 'astro/loaders';

const authors = defineCollection({
  loader: file('src/data/authors.json'),
  schema: z.object({ name: z.string() }),
});
```

Supports a custom `parser` for non-standard formats (e.g., CSV).

### Custom Loaders

Load from any source (APIs, databases, CMSes):

```typescript
const products = defineCollection({
  loader: async () => {
    const response = await fetch('https://api.example.com/products');
    const data = await response.json();
    return data.map((product: any) => ({
      id: product.id,
      ...product,
    }));
  },
  schema: z.object({ name: z.string(), price: z.number() }),
});
```

### Object Loaders (Advanced)

For full control with incremental updates, caching, and file watching:

```typescript
import type { Loader } from 'astro/loaders';

function myLoader(options: { url: string }): Loader {
  return {
    name: 'my-loader',
    load: async ({ store, meta, logger }) => {
      const lastModified = meta.get('lastModified');
      const data = await fetchData(options.url, lastModified);

      store.clear();
      for (const item of data) {
        store.set({ id: item.id, data: item });
      }

      meta.set('lastModified', new Date().toISOString());
    },
  };
}
```

### LoaderContext helpers

The `load` method receives a `LoaderContext` with useful helpers:

| Helper | Description |
|--------|-------------|
| `store` | The data store (`clear()`, `set()`, `get()`, `values()`) |
| `meta` | Key/value metadata store for incremental updates |
| `parseData({ id, data })` | Validates data against the collection schema |
| `generateDigest(data)` | Generates a non-cryptographic content digest to track changes |
| `watch` | File watcher for dev-mode reloads |
| `logger` | Logger scoped to the loader (`logger.info(...)`) |
| `config` | The resolved Astro config |

```typescript
import type { Loader } from 'astro/loaders';

function feedLoader({ url }: { url: string }): Loader {
  return {
    name: 'feed-loader',
    load: async ({ store, logger, parseData, generateDigest }) => {
      logger.info('Loading posts');
      const feed = await loadFeed(url);
      store.clear();

      for (const item of feed.items) {
        const data = await parseData({ id: item.guid, data: item });
        const digest = generateDigest(data);
        store.set({ id: item.guid, data, digest });
      }
    },
  };
}
```

## Live Loaders (Astro 6+)

Live collections fetch data fresh on every request — no data store to update. Use for real-time data.

### Configuration file: `src/live.config.ts`

Live collections are defined in `src/live.config.ts` (separate from `src/content.config.ts`) using `defineLiveCollection()` from `astro:content`:

```typescript
// src/live.config.ts
import { defineLiveCollection } from 'astro:content';
import { z } from 'astro/zod';
import { productLoader } from './loaders/product-loader';

const products = defineLiveCollection({
  loader: productLoader({ apiKey: process.env.STORE_API_KEY }),
  // Optional: a Zod schema takes precedence over the loader's types
  schema: z.object({
    id: z.string(),
    name: z.string(),
    price: z.number(),
  }),
});

export const collections = { products };
```

> **Note:** Live collections require an adapter (on-demand rendering). There are no built-in live loaders — you must create a custom one.

### Creating a live loader

A live loader implements `loadCollection()` and `loadEntry()` (instead of `load`) and returns data directly. Use the generic type `LiveLoader<TData, TEntryFilter, TCollectionFilter, TError>`:

```typescript
// src/loaders/product-loader.ts
import type { LiveLoader } from 'astro/loaders';

interface Product {
  id: string;
  name: string;
  price: number;
}

interface EntryFilter {
  id: string;
}

interface CollectionFilter {
  category?: string;
}

export function productLoader(config: { apiKey: string }): LiveLoader<
  Product,
  EntryFilter,
  CollectionFilter
> {
  return {
    name: 'product-loader',
    loadCollection: async ({ filter }) => {
      try {
        const data = await fetchProducts(config.apiKey, filter);
        return {
          entries: data.map((p) => ({ id: p.sku, data: p })),
        };
      } catch (error) {
        return { error: new Error('Failed to load products', { cause: error }) };
      }
    },
    loadEntry: async ({ filter }) => {
      const product = await fetchProduct(config.apiKey, filter.id);
      if (!product) return undefined;
      return { id: product.sku, data: product };
    },
  };
}
```

Generic parameters (in order): `TData` (entry data, default `Record<string, unknown>`), `TEntryFilter` (default `never`), `TCollectionFilter` (default `never`), `TError` (custom `Error` subclass, default `Error`).

### Querying live collections

Query with `getLiveCollection()` and `getLiveEntry()`:

```astro
---
import { getLiveCollection, getLiveEntry } from 'astro:content';

const { entries, error } = await getLiveCollection('products');
const { entry, error: entryError } = await getLiveEntry('products', 'sku-123');
---
```

### Live loader errors

Errors thrown in a live loader are caught and wrapped in `LiveCollectionError`. Astro also generates:

- `LiveEntryNotFoundError` — when `loadEntry` returns `undefined`
- `LiveCollectionValidationError` — when data does not match the collection schema
- `LiveCollectionCacheHintError` — when a loader returns an invalid cache hint

```astro
---
import { LiveCollectionValidationError } from 'astro/content/runtime';
import { getLiveEntry } from 'astro:content';

const { entry, error } = await getLiveEntry('products', '123');
if (LiveCollectionValidationError.is(error)) {
  console.error(error.message);
  return Astro.rewrite('/500');
}
---
```

### Caching live data (Astro 7)

Live loaders can return a `cacheHint` (with `tags` and `lastModified`). Pass it to `Astro.cache.set()` to apply the loader's recommended caching strategy:

```astro
---
import { getLiveEntry } from 'astro:content';

const { entry, error, cacheHint } = await getLiveEntry('products', Astro.params.id);
if (error) return Astro.redirect('/404');

if (cacheHint) {
  Astro.cache.set(cacheHint);
}
Astro.cache.set({ maxAge: 300 });
---
<h1>{entry.data.name}</h1>
```

You can also pass a `LiveDataEntry` directly to `Astro.cache.set()` to extract its `cacheHint` automatically, and invalidate by entry with `context.cache.invalidate(entry)`.

## Collection Types (Legacy — Astro 4)

### Content Collections (`type: 'content'`)

For Markdown and MDX files with frontmatter:

```markdown
---
title: "My First Post"
pubDate: 2024-01-15
author: "john"
---

# Hello World

This is my first blog post.
```

### Data Collections (`type: 'data'`)

For JSON or YAML data files:

```json
// src/content/authors/john.json
{
  "name": "John Doe",
  "email": "john@example.com",
  "bio": "A passionate writer",
  "avatar": "https://example.com/avatar.jpg"
}
```

## Schema Validation with Zod

### Common Field Types

```typescript
import { z } from 'astro/zod'; // Astro 5+
// import { z } from 'astro:content'; // Legacy (Astro 4)

const schema = z.object({
  // Strings
  title: z.string(),
  slug: z.string().regex(/^[a-z0-9-]+$/),

  // Numbers
  order: z.number().int().positive(),
  rating: z.number().min(0).max(5),

  // Booleans
  featured: z.boolean().default(false),

  // Dates
  pubDate: z.coerce.date(), // Converts strings to Date

  // Arrays
  tags: z.array(z.string()),
  categories: z.array(z.enum(['tech', 'life', 'travel'])),

  // Objects
  author: z.object({
    name: z.string(),
    email: z.string().email(),
  }),

  // Enums
  status: z.enum(['draft', 'published', 'archived']),

  // Optional fields
  description: z.string().optional(),
  image: z.string().url().optional(),

  // Default values
  views: z.number().default(0),

  // Unions
  media: z.union([
    z.object({ type: z.literal('image'), src: z.string() }),
    z.object({ type: z.literal('video'), url: z.string() }),
  ]),
});
```

### Image Schema

```typescript
// Astro 5+ with loader
import { defineCollection } from 'astro:content';
import { glob } from 'astro/loaders';
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.md' }),
  schema: ({ image }) => z.object({
    title: z.string(),
    cover: image(), // Validates and optimizes images
    coverAlt: z.string(),
  }),
});
```

### Reference Other Collections

```typescript
import { defineCollection, reference } from 'astro:content';
import { z } from 'astro/zod';

const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.md' }),
  schema: z.object({
    title: z.string(),
    author: reference('authors'), // References authors collection
    relatedPosts: z.array(reference('blog')).optional(),
  }),
});
```

## Querying Collections

### getCollection()

```astro
---
import { getCollection } from 'astro:content';

// Get all entries
const allPosts = await getCollection('blog');

// Filter entries
const publishedPosts = await getCollection('blog', ({ data }) => {
  return data.draft !== true;
});

// Filter by date
const recentPosts = await getCollection('blog', ({ data }) => {
  return data.pubDate > new Date('2024-01-01');
});

// Sort entries
const sortedPosts = (await getCollection('blog'))
  .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---

<ul>
  {sortedPosts.map((post) => (
    <li>
      <a href={`/blog/${post.slug}`}>{post.data.title}</a>
      <time>{post.data.pubDate.toLocaleDateString()}</time>
    </li>
  ))}
</ul>
```

### getEntry()

```astro
---
import { getEntry } from 'astro:content';

// Get single entry by collection and slug
const post = await getEntry('blog', 'my-first-post');

// Get single entry by reference
const author = await getEntry(post.data.author);

// Check if entry exists
if (!post) {
  return Astro.redirect('/404');
}
---

<article>
  <h1>{post.data.title}</h1>
  <p>By {author.data.name}</p>
</article>
```

### getEntries()

```astro
---
import { getEntry, getEntries } from 'astro:content';

const post = await getEntry('blog', 'my-post');

// Get multiple referenced entries
const relatedPosts = await getEntries(post.data.relatedPosts);
---
```

## Rendering Content

### Astro 5+ (import `render` from `astro:content`)

```astro
---
import { getEntry, render } from 'astro:content';

const post = await getEntry('blog', 'my-post');
const { Content, headings } = await render(post);
---

<article>
  <h1>{post.data.title}</h1>

  <!-- Table of contents from headings -->
  <nav>
    {headings.map((h) => (
      <a href={`#${h.slug}`} style={`margin-left: ${h.depth * 10}px`}>
        {h.text}
      </a>
    ))}
  </nav>

  <!-- Rendered content -->
  <Content />
</article>
```

### Legacy (Astro 4)

```astro
---
const post = await getEntry('blog', 'my-post');
const { Content, headings } = await post.render();
---
```

## Dynamic Routes with Collections

```astro
---
// src/pages/blog/[...slug].astro
import { getCollection, render } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---

<article>
  <h1>{post.data.title}</h1>
  <Content />
</article>
```

> **Note:** In Astro 5+, use `post.id` instead of `post.slug` for routing params.

## Expert Guidance

### Loader Selection Decision Tree

```
Local markdown/MDX files → glob() loader
Single JSON/YAML data file → file() loader
Remote API/CMS data at build → Custom async loader function
Remote data fresh per-request → Live Loader (Astro 6+)
```

### Performance: `retainBody` for Large Sites

For sites with >1000 content entries where you only need frontmatter data (e.g., listing pages, tag indexes), disable body storage:

```typescript
const blog = defineCollection({
  loader: glob({
    base: './src/content/blog',
    pattern: '**/*.md',
    retainBody: false, // Significantly reduces data store size
  }),
  schema: z.object({ title: z.string(), pubDate: z.coerce.date() }),
});
```

Only use `retainBody: false` for collections where you don't call `render()`. If you need to render content on detail pages, keep the default (`true`).

### Migration Pitfalls (Astro 4 → 5)

| What changed | Old (Astro 4) | New (Astro 5+) |
|---|---|---|
| Config file | `src/content/config.ts` | `src/content.config.ts` |
| Collection type | `type: 'content'` / `type: 'data'` | `loader: glob(...)` / `loader: file(...)` |
| Zod import | `import { z } from 'astro:content'` | `import { z } from 'astro/zod'` |
| Rendering | `const { Content } = await entry.render()` | `import { render } from 'astro:content'; await render(entry)` |
| Route param | `post.slug` | `post.id` |

**The most common migration bug:** importing `z` from `astro:content` instead of `astro/zod`. This still compiles but can cause subtle type mismatches with the new Content Layer API.

### Schema Design Anti-Patterns

- **`z.date()` for frontmatter dates** → Always use `z.coerce.date()`. YAML/frontmatter dates arrive as strings; `z.date()` rejects them silently
- **Missing `.default()` on optional booleans** → `draft: z.boolean().optional()` means `undefined`, not `false`. Use `z.boolean().default(false)` so filtering logic works correctly
- **String paths for images** → Use the `image()` schema helper to enable Astro's image optimization pipeline. String URLs bypass optimization entirely

## Best Practices

1. **Always define schemas** - Type safety and validation
2. **Use `z.coerce.date()`** - Handles string dates from frontmatter
3. **Set sensible defaults** - `.default()` reduces required fields
4. **Filter drafts in production** - Don't expose unpublished content
5. **Use references for relationships** - Type-safe cross-collection links
6. **Keep slugs URL-friendly** - Validate with regex
7. **Use image schema for images** - Enables optimization
