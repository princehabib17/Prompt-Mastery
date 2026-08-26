---
name: matt-pocock-typescript
description: >
  Write and review TypeScript with Matt Pocock's opinionated type-safety
  taste: types inferred rather than annotated, `unknown` narrowed instead of
  `any`, discriminated unions instead of flag soup, `as const` literal unions
  instead of `enum`, generics inferred from usage instead of hand-supplied,
  exhaustive switches with a `never` check instead of partial handling. Use
  this whenever writing new TypeScript, reviewing a TypeScript diff or PR,
  designing a type for application state, refactoring a function's generics,
  or when the user asks for "clean TypeScript", "type-safe" code, help with
  a gnarly generic, or how to model a piece of state in types. Also trigger
  when a request just says "make this TypeScript better" or "Pocock-style"
  with no other detail.
---

# matt-pocock-typescript

A checklist-driven taste for TypeScript, not a linter config. The goal in
every case is the same: make illegal states unrepresentable, and let the
compiler do the narrowing work a human would otherwise do by hand.

## Core moves, in priority order

### 1. Let inference do the work

Don't annotate what TypeScript can already work out. An explicit annotation
that just repeats the inferred type is noise, and worse, it can silently
hide a real mismatch that inference would have caught.

```ts
// Avoid — annotation adds nothing, and can drift from the real return type
const getUser = (id: string): User => {
  return db.users.find(id);
};

// Prefer — let the return type follow the implementation
const getUser = (id: string) => {
  return db.users.find(id);
};
```

Annotate at boundaries where inference has nothing to go on: function
parameters, and exported function return types on public APIs (so a change
inside the function can't silently change its public contract).

### 2. `unknown`, not `any`

`any` disables the type checker everywhere it flows. `unknown` forces a
narrowing step before use, which is exactly the point.

```ts
// Avoid
function parse(input: any) {
  return input.data.value; // no safety, no errors, ever
}

// Prefer
function parse(input: unknown) {
  if (typeof input !== "object" || input === null || !("data" in input)) {
    throw new Error("Invalid input");
  }
  // narrowed from here down
}
```

If narrowing by hand gets repetitive, that's a sign to reach for a schema
library (Zod, Valibot, ArkType) rather than writing ad hoc type guards for
every shape.

### 3. Discriminated unions over flag soup

A handful of independent booleans/optionals describing "what state is this
in" almost always has invalid combinations the type system can't rule out.
A discriminated union makes every valid state explicit and every invalid
one unrepresentable.

```ts
// Avoid — isLoading: true, error: "x", data: {...} all at once compiles fine
type State = {
  isLoading: boolean;
  error?: string;
  data?: User;
};

// Prefer — exactly one shape is ever true at a time
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "error"; error: string }
  | { status: "success"; data: User };
```

Once state is modeled this way, a `switch` on the discriminant reads like
documentation, and the compiler complains the moment a case is missed.

### 4. No `enum`; use `as const` + a union type

TypeScript `enum` has real footguns (numeric enums are bidirectionally
mapped, `const enum` doesn't work everywhere, string enums aren't
structurally compatible with plain string literals). A `const` object plus
a derived union gets the same ergonomics without the sharp edges.

```ts
// Avoid
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

// Prefer
const Direction = {
  Up: "up",
  Down: "down",
  Left: "left",
  Right: "right",
} as const;

type Direction = (typeof Direction)[keyof typeof Direction];
```

If there's no need for the runtime object at all, a bare union of string
literals (`type Direction = "up" | "down" | "left" | "right"`) is even
simpler — reach for the `as const` object only when call sites want
autocomplete on `Direction.Up` or need to iterate the values at runtime.

### 5. Infer generics from usage, don't hand-supply them

A generic that has to be manually specified at every call site is a sign
the function isn't inferring what it should. Design the function signature
so TypeScript can pick up the type parameter from an argument.

```ts
// Avoid — caller must remember to write <User> every time
function createStore<T>(): Store<T> {
  /* ... */
}
const store = createStore<User>();

// Prefer — T is inferred from the initial value
function createStore<T>(initial: T): Store<T> {
  /* ... */
}
const store = createStore<User>({ id: "1", name: "Bob" }); // or just infer it
```

When a generic truly can't be inferred from any argument (e.g. a factory
with no input shaped like the output), that's fine — it's just a signal
that a manual type argument is the correct, unavoidable call-site cost,
not something to work around with `any`.

### 6. Favor total functions and exhaustive switches

A function that only handles some of its inputs pushes the missing cases
onto every caller. Prefer handling every case explicitly, and use a
`never`-typed exhaustiveness check so adding a new variant to a union is a
compile error everywhere it isn't handled, not a runtime surprise.

```ts
function getLabel(status: State["status"]) {
  switch (status) {
    case "idle":
      return "Idle";
    case "loading":
      return "Loading…";
    case "error":
      return "Something went wrong";
    case "success":
      return "Done";
    default:
      status satisfies never; // add a new status → this line stops compiling
      throw new Error(`Unhandled status: ${status}`);
  }
}
```

Prefer `satisfies never` (or `const _exhaustive: never = status`) over
just letting the switch fall through silently.

### 7. Compose small utility types instead of one mega-type

A single type with several layers of nested generics, conditional types,
and mapped types is hard to read and hard to debug when it fails. Break it
into named, individually-understandable pieces — each one testable and
reusable on its own — and compose them.

```ts
// Avoid — one dense type doing three jobs at once
type ApiResponse<T> = T extends { id: string }
  ? { data: T; meta: { id: string; cached: boolean } }
  : { data: T; meta: { cached: boolean } };

// Prefer — name the pieces
type WithId = { id: string };
type Meta<T> = T extends WithId ? { id: string; cached: boolean } : { cached: boolean };
type ApiResponse<T> = { data: T; meta: Meta<T> };
```

Only reach for the deep-generic version when the simpler composed version
genuinely can't express what's needed — and even then, keep each named
piece under a few lines.

### 8. No unnecessary classes

A class is the right tool when there's real state that needs encapsulating
alongside behavior that mutates it (e.g. a stateful connection, a cache
with eviction logic). If a "class" is really just a bag of pure functions,
or a single method wrapping some data, it's simpler as a plain object or a
function.

```ts
// Avoid — no real state, no inheritance, just namespacing
class MathUtils {
  static add(a: number, b: number) {
    return a + b;
  }
}

// Prefer
export const add = (a: number, b: number) => a + b;
```

### 9. Naming

- Types and interfaces: `PascalCase`, no `I` prefix (`User`, not `IUser`).
- Names describe what the thing *is*, not an abbreviation of it
  (`UserSettings`, not `UsrCfg`).
- Prefer `type` for unions, intersections, and anything mapped/conditional;
  `interface` is fine for plain object shapes meant to be extended, but
  either is acceptable as long as it's consistent within a file.

## Review checklist

When reviewing TypeScript rather than writing it, scan for:

- [ ] Explicit type annotations that just restate an inferred type
- [ ] Any `any` — could it be `unknown` + a narrowing check or schema parse?
- [ ] Optional/boolean fields that model mutually exclusive states — could
      this be a discriminated union instead?
- [ ] Any `enum` — could it be a literal union or `as const` object?
- [ ] Generic type parameters supplied manually at call sites that could
      instead be inferred from an argument
- [ ] `switch` statements over a union missing a `never` exhaustiveness
      check in the default case
- [ ] One large, deeply-nested generic type that could be decomposed into
      several small named ones
- [ ] A `class` with no real internal state or mutation — could it be
      plain functions?
- [ ] `interface`/`type` names with an `I` prefix or abbreviated names

Don't rewrite for the sake of it — apply these where they genuinely
simplify the code or close a real type-safety gap, not as a mechanical
pass over every line.
