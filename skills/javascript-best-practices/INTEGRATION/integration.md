# Integration with Node.js Skills

## Overview

The `javascript-best-practices` skill complements the existing Node.js skills in your workspace. Here's how they work together.

## Skill Comparison

| Skill | Focus | Use When |
|-------|-------|----------|
| `javascript-best-practices` | JS/TS language fundamentals | Writing any JS/TS code |
| `nodejs-best-practices` | Node.js philosophy & decisions | Making architecture decisions |
| `nodejs-backend-patterns` | Express/Fastify patterns | Building API backends |

## When to Use Each

### JavaScript Best Practices

- Writing business logic in any environment
- TypeScript type design
- Clean code and naming conventions
- General JavaScript patterns

```typescript
// Focus: language-level best practices
function processUserData(user: User): ProcessedUser {
  return {
    ...user,
    displayName: user.name.toUpperCase(),
    processed: true
  };
}
```

### Node.js Best Practices

- Choosing between frameworks (Express vs Fastify vs Hono)
- Runtime decisions (Node.js vs Bun vs Deno)
- Architecture patterns (layered architecture, error handling)
- Security principles

```typescript
// Focus: framework and runtime decisions
// See nodejs-best-practices for framework selection decision tree
```

### Node.js Backend Patterns

- Express/Fastify middleware
- API route handlers
- Database integration patterns
- Authentication/authorization

```typescript
// Focus: HTTP server patterns
// See nodejs-backend-patterns for Express/Fastify middleware patterns
```

## Integration Examples

### Example: Building an API Endpoint

**Skill Stack:**
1. `javascript-best-practices` - TypeScript types, clean functions
2. `nodejs-backend-patterns` - Express route structure, middleware
3. `nodejs-best-practices` - Error handling strategy, security

```typescript
// 1. Define types (javascript-best-practices)
interface CreateUserDto {
  name: string;
  email: string;
  role?: 'admin' | 'user';
}

interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
  createdAt: Date;
}

// 2. Business logic with clean functions (javascript-best-practices)
function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

function createUser(dto: CreateUserDto): User {
  if (!validateEmail(dto.email)) {
    throw new ValidationError('Invalid email format');
  }

  return {
    id: crypto.randomUUID(),
    name: dto.name,
    email: dto.email,
    role: dto.role ?? 'user',
    createdAt: new Date()
  };
}

// 3. Express route handler (nodejs-backend-patterns)
import { Router, Request, Response, NextFunction } from 'express';

const router = Router();

const asyncHandler = (fn: (req: Request, res: Response, next: NextFunction) => Promise<void>) =>
  (req: Request, res: Response, next: NextFunction) =>
    Promise.resolve(fn(req, res, next)).catch(next);

router.post('/users', asyncHandler(async (req: Request, res: Response) => {
  const dto: CreateUserDto = req.body;
  const user = createUser(dto);

  res.status(201).json(user);
}));

// 4. Error handling (nodejs-best-practices)
class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}
```

### Example: Async Data Processing

**Skill Stack:**
1. `javascript-best-practices` - Async/await patterns, error handling
2. `nodejs-best-practices` - Event loop awareness, worker threads

```typescript
// 1. Async patterns (javascript-best-practices)
async function processWithRetry<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  let lastError: Error;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error as Error;
      if (i < maxRetries - 1) {
        await delay(Math.pow(2, i) * 1000); // exponential backoff
      }
    }
  }

  throw lastError!;
}

// 2. CPU-intensive work awareness (nodejs-best-practices)
// For CPU-bound work, offload to worker threads
import { Worker } from 'worker_threads';

function processInWorker<T>(data: unknown): Promise<T> {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./processor.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}
```

## Decision Flowchart

```
What are you working on?
│
├── Frontend JavaScript/TypeScript
│   └── Use javascript-best-practices
│
├── Backend Node.js API
│   ├── Choosing framework
│   │   └── See nodejs-best-practices (Framework Selection)
│   │
│   ├── Express/Fastify patterns
│   │   └── See nodejs-backend-patterns
│   │
│   └── Business logic / TypeScript
│       └── Use javascript-best-practices
│
└── Architecture Decisions
    └── See nodejs-best-practices
```

## Combining Skills

### For New Projects

1. Start with `nodejs-best-practices` for framework/runtime decisions
2. Use `nodejs-backend-patterns` for API structure
3. Apply `javascript-best-practices` for code quality

### For Code Review

1. Check TypeScript types with `javascript-best-practices`
2. Verify patterns with `javascript-best-practices`
3. Review architecture with `nodejs-best-practices`

### For Learning

1. `javascript-best-practices` - Foundation in JS/TS
2. `nodejs-best-practices` - Backend-specific knowledge
3. `nodejs-backend-patterns` - Practical implementation

## Summary

- **`javascript-best-practices`**: Language-level quality
- **`nodejs-best-practices`**: Philosophy and decisions
- **`nodejs-backend-patterns`**: HTTP framework implementation

Together they provide comprehensive coverage from language fundamentals to production backend patterns.