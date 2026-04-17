---
name: typescript-best-practices
description: "Guide AI agents through TypeScript coding best practices including type safety, error handling, code organization, architecture patterns, and SOLID design. This skill should be used when generating TypeScript code, reviewing TypeScript files, creating new TypeScript modules, refactoring JavaScript to TypeScript, or when the user asks about TypeScript patterns, types, or coding standards. Keywords: typescript, types, coding standards, best practices, type safety, generics, architecture, refactoring, solid."
license: MIT
compatibility: Applicable to any TypeScript codebase.
metadata:
  author: agent-skills
  version: "1.0"
  type: utility
  mode: assistive
  domain: development
---

# TypeScript Best Practices

Guide AI agents in writing high-quality TypeScript code. This skill provides coding standards, architecture patterns, and practical reference guidance.

## When to Use This Skill

Use this skill when:

- Generating new TypeScript code
- Reviewing TypeScript files for quality issues
- Creating new modules, services, or components
- Refactoring JavaScript to TypeScript
- Answering questions about TypeScript patterns or types
- Designing APIs or interfaces

Do NOT use this skill when:

- Working with pure JavaScript (no TypeScript)
- Debugging runtime errors (use debugging tools)
- Framework-specific patterns (React, Vue, etc. - use framework skills)

## Core Principles

### 1. Type Safety First

Maximize compile-time error detection:

```typescript
// Prefer unknown over any for unknown types
function processInput(data: unknown): string {
  if (typeof data === "string") return data;
  if (typeof data === "number") return String(data);
  throw new Error("Unsupported type");
}

// Explicit return types for public APIs
export function calculateTotal(items: ReadonlyArray<Item>): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// Use const assertions for literal types
const CONFIG = {
  mode: "production",
  version: 1,
} as const;
```

### 2. Immutability by Default

Prevent accidental mutations:

```typescript
// Use readonly for object properties
interface User {
  readonly id: string;
  readonly email: string;
  name: string; // Only mutable if intentional
}

// Use ReadonlyArray for collections
function processItems(items: ReadonlyArray<Item>): ReadonlyArray<Result> {
  return items.map(transform);
}

// Prefer spreading over mutation
function updateUser(user: User, name: string): User {
  return { ...user, name };
}
```

### 3. Error Handling with Types

Use the type system for error handling:

```typescript
// Result type for recoverable errors
type Result<T, E = Error> =
  | { success: true; value: T }
  | { success: false; error: E };

// Typed error classes
class ValidationError extends Error {
  constructor(
    message: string,
    readonly field: string,
    readonly code: string,
  ) {
    super(message);
    this.name = "ValidationError";
  }
}

// Function with Result return type
function parseConfig(input: string): Result<Config, ValidationError> {
  try {
    const data = JSON.parse(input);
    if (!isValidConfig(data)) {
      return {
        success: false,
        error: new ValidationError("Invalid config", "root", "INVALID_FORMAT"),
      };
    }
    return { success: true, value: data };
  } catch {
    return {
      success: false,
      error: new ValidationError("Parse failed", "root", "PARSE_ERROR"),
    };
  }
}
```

### 4. Code Organization

Structure code for maintainability:

```typescript
// One concept per file
// user.ts - User type and related utilities
export interface User {
  readonly id: string;
  readonly email: string;
  readonly createdAt: Date;
}

export function createUser(email: string): User {
  return {
    id: crypto.randomUUID(),
    email,
    createdAt: new Date(),
  };
}

// Explicit exports (no barrel file wildcards)
// index.ts
export { User, createUser } from "./user.ts";
export { validateEmail } from "./validation.ts";
```

### 5. SOLID for TypeScript Implementation

Apply SOLID as implementation rules, not abstract theory:

```typescript
interface BatchStore {
  get(key: string): Promise<PlaybackBatch | null>;
  set(key: string, batch: PlaybackBatch): Promise<void>;
}

interface BatchReporter {
  report(batch: PlaybackBatch): Promise<boolean>;
}

interface PlaybackEvent {
  readonly creativeId: string;
  readonly playedAt: string;
}

interface PlaybackBatch {
  readonly deviceId: string;
  readonly events: ReadonlyArray<PlaybackEvent>;
}

class PlaybackBatchService {
  constructor(
    private readonly store: BatchStore,
    private readonly reporter: BatchReporter,
    private readonly maxBatchSize: number,
  ) {}

  async append(deviceId: string, event: PlaybackEvent): Promise<void> {
    const key = this.makeStorageKey(deviceId);
    const current =
      (await this.store.get(key)) ?? this.makeEmptyBatch(deviceId);
    const updated = this.addEvent(current, event);
    const maybeFlushed = await this.flushIfNeeded(updated);
    await this.store.set(key, maybeFlushed);
  }

  private makeStorageKey(deviceId: string): string {
    return `playback:${deviceId}`;
  }

  private makeEmptyBatch(deviceId: string): PlaybackBatch {
    return { deviceId, events: [] };
  }

  private addEvent(batch: PlaybackBatch, event: PlaybackEvent): PlaybackBatch {
    return { ...batch, events: [...batch.events, event] };
  }

  private async flushIfNeeded(batch: PlaybackBatch): Promise<PlaybackBatch> {
    if (batch.events.length < this.maxBatchSize) return batch;
    const reported = await this.reporter.report(batch);
    return reported ? { ...batch, events: [] } : batch;
  }
}

// Composition root: wire concrete implementations at the boundary.
const playbackService = new PlaybackBatchService(
  new IndexedDbBatchStore(indexedDbClient),
  new HttpBatchReporter(httpClient),
  100,
);
```

## Quick Reference

| Category       | Prefer                                                          | Avoid                                         |
| -------------- | --------------------------------------------------------------- | --------------------------------------------- |
| Unknown types  | `unknown`                                                       | `any`                                         |
| Collections    | `ReadonlyArray<T>`                                              | `T[]` for inputs                              |
| Objects        | `Readonly<T>`                                                   | Mutable by default                            |
| Null checks    | Optional chaining `?.`                                          | `!= null`                                     |
| Type narrowing | Type guards                                                     | `as` assertions                               |
| Return types   | Explicit on exports                                             | Inferred on exports                           |
| Enums          | String literal unions                                           | Numeric enums                                 |
| Imports        | Named imports                                                   | Default imports                               |
| Errors         | Result types                                                    | Throwing for flow control                     |
| Design         | SOLID + interface boundaries                                    | God classes + concrete coupling               |
| Class patterns | `static` for type-level helpers; `abstract` for shared behavior | Mutable static state; inheritance-only design |
| Loops          | `for...of`, `.map()`                                            | `for...in` on arrays                          |

## Code Generation Guidelines

When generating TypeScript code, follow these patterns:

### Module Structure

```typescript
/**
 * Module description
 * @module module-name
 */

// === Types ===
export interface ModuleOptions {
  readonly setting: string;
}

export interface ModuleResult {
  readonly data: unknown;
}

// === Constants ===
const DEFAULT_OPTIONS: ModuleOptions = {
  setting: "default",
};

// === Implementation ===
export function processData(
  input: unknown,
  options: Partial<ModuleOptions> = {},
): ModuleResult {
  const opts = { ...DEFAULT_OPTIONS, ...options };
  // Implementation
  return { data: input };
}
```

### Function Design

Use `function` declarations for named top-level or exported APIs. Use arrow functions for inline callbacks and when lexical `this` binding is required.

```typescript
// Pure functions preferred
function transform(input: Input): Output {
  // No side effects, same input = same output
  return { ...input, processed: true };
}

// Explicit parameter types
function fetchUser(id: string, options?: FetchOptions): Promise<User> {
  // Implementation
}

// Use function overloads for complex signatures
function parse(input: string): ParsedData;
function parse(input: Buffer): ParsedData;
function parse(input: string | Buffer): ParsedData {
  // Implementation
}

// Use arrows for short callbacks
const names = users.map((user) => user.name);
```

### Interface Design

```typescript
// Prefer interfaces for object shapes
interface UserData {
  readonly id: string;
  readonly email: string;
}

// Use type for unions and intersections
type UserRole = "admin" | "user" | "guest";
type AdminUser = UserData & { readonly role: "admin" };

// Document with JSDoc
/**
 * Configuration for the API client
 * @property baseUrl - The base URL for API requests
 * @property timeout - Request timeout in milliseconds
 */
interface ApiConfig {
  readonly baseUrl: string;
  readonly timeout?: number;
}
```

### Class Design (`static` and `abstract`)

Use `static` for behavior that belongs to the type itself and does not require instance state.
Use `abstract` classes only when you need shared behavior plus extension points.

```typescript
class UserId {
  private static readonly pattern = /^usr_[a-z0-9]{12}$/;

  private constructor(readonly value: string) {}

  static from(raw: string): UserId {
    if (!UserId.pattern.test(raw)) {
      throw new Error(`Invalid user id: ${raw}`);
    }
    return new UserId(raw);
  }
}

type SaveResult = { success: true } | { success: false; reason: string };

abstract class BaseRepository<T extends { readonly id: string }> {
  async save(entity: T): Promise<SaveResult> {
    if (!this.isValid(entity)) {
      return { success: false, reason: "Invalid entity" };
    }

    return this.persist(entity);
  }

  protected isValid(entity: T): boolean {
    return entity.id.length > 0;
  }

  protected abstract persist(entity: T): Promise<SaveResult>;
}
```

## Common Anti-Patterns

Avoid these patterns when generating code:

| Anti-Pattern              | Problem                 | Solution                                   |
| ------------------------- | ----------------------- | ------------------------------------------ |
| `any` type                | Disables type checking  | Use `unknown` and narrow                   |
| `as` assertions           | Runtime errors          | Use type guards                            |
| Non-null `!`              | Null pointer errors     | Optional chaining `?.`                     |
| Mutable params            | Unexpected mutations    | `Readonly<T>`                              |
| Magic strings             | Typos, no autocomplete  | String literal types                       |
| God classes               | Hard to test/maintain   | Single responsibility                      |
| Circular deps             | Build/runtime issues    | Dependency inversion                       |
| Index signatures          | Lose type info          | Explicit properties                        |
| Mutable static state      | Hidden global behavior  | `static readonly` or module constants      |
| Abstract-only passthrough | Unnecessary inheritance | Interface first, abstract for shared logic |

See `references/anti-patterns/common-mistakes.md` for detailed examples.

## Additional Resources

### Type System Deep Dives

- `references/type-system/advanced-types.md` - Generics, conditional types, mapped types
- `references/type-system/type-guards.md` - Type narrowing techniques
- `references/type-system/utility-types.md` - Built-in utility types

### Pattern Guides

- `references/patterns/error-handling.md` - Result types, typed errors
- `references/patterns/async-patterns.md` - Async/await best practices
- `references/patterns/functional-patterns.md` - Immutability, composition
- `references/patterns/module-patterns.md` - Exports, dependency injection
- `references/patterns/solid-principles.md` - SOLID principles for TypeScript implementation

### Architecture

- `references/architecture/api-design.md` - Interface design, versioning
