# JavaScript Best Practices - Usage Guide

## Overview

This skill provides best practices for JavaScript and TypeScript development, covering naming conventions, patterns, clean code principles, and modern ES2025+ practices.

## File Structure

```
javascript-best-practices/
├── SKILL.md                    # This file - skill index
├── README.md                   # This guide
├── FUNDAMENTALS/
│   ├── naming.md              # Naming conventions
│   ├── typescript-essentials.md
│   └── clean-code.md
├── PATTERNS/
│   ├── functions.md
│   ├── async.md
│   ├── state.md
│   └── design-patterns.md
└── INTEGRATION/
    └── integration.md
```

## How to Use

### 1. Consultation Mode

When working on JS/TS code, load this skill to ensure you're following best practices:

```
Load: javascript-best-practices
```

### 2. Quick Reference

For specific topics, reference the relevant file:

| Topic | File |
|-------|------|
| Variable naming | `FUNDAMENTALS/naming.md` |
| TypeScript types | `FUNDAMENTALS/typescript-essentials.md` |
| SOLID principles | `FUNDAMENTALS/clean-code.md` |
| Function composition | `PATTERNS/functions.md` |
| Async patterns | `PATTERNS/async.md` |
| State management | `PATTERNS/state.md` |
| Design patterns | `PATTERNS/design-patterns.md` |

### 3. Integration with Other Skills

This skill works best combined with:

- **`nodejs-best-practices`** - For backend Node.js decisions
- **`nodejs-backend-patterns`** - For Express/Fastify API patterns
- **`security-best-practices`** - For security-related JS patterns

## Content Format

Each file follows a consistent structure:

- **Concept** - Clear explanation of the principle/pattern
- **✅ Good** - Recommended code example
- **❌ Bad** - Code to avoid
- **Note** - Additional context when relevant

## TypeScript Coverage

This skill covers TypeScript including:

- Basic types and inference
- Generics and constraints
- Interfaces vs Type Aliases
- Union types and discriminated unions
- Type guards and narrowing
- Utility types
- Mapped types

## JavaScript Coverage

Modern JavaScript (ES2025+) including:

- Arrow functions and implicit returns
- Destructuring and spread operators
- Optional chaining and nullish coalescing
- Top-level await
- Private class fields
- Pattern matching (experimental)

## Contributing

When adding content:

1. Follow the concept + examples format
2. Include both ✅ Good and ❌ Bad examples
3. Add Notes for important context
4. Keep examples concise (5-15 lines max)
5. Mark experimental features accordingly