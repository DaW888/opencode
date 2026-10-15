---
name: typescript
description: Conventions for writing, editing, reviewing, and testing TypeScript. Load before reading or changing any .ts, .mts, or .cts file, including TypeScript tests and Node.js backend code.
---

# TypeScript

Repo conventions (tsconfig, lint config, existing code style, AGENTS.md) take precedence over this skill. If the repo clearly does something differently, follow the repo.

## Type safety

- Never use `any`. Use `unknown` and narrow it with `typeof`, `instanceof`, `in`, or type guards.
- Avoid type assertions (`as`) and non-null assertions (`!`); narrow instead. `as const` and `satisfies` are fine.
- Don't use `Function`, `Object`, `{}`, or `object` as catch-all types. Write the actual signature or shape.
- Treat external data (`JSON.parse`, `fetch` responses, env vars, file contents) as `unknown` until validated. Use the repo's schema library (zod, valibot, etc.) if present, otherwise type guards.
- Treat array and record index access as possibly `undefined`, even without `noUncheckedIndexedAccess`.
- Never use `@ts-ignore` or `@ts-nocheck`. If suppression is truly unavoidable, use `@ts-expect-error` with a reason on the same line.
- Exported functions declare explicit return types.
- Don't loosen `tsconfig`, ESLint/Biome config, or add disable comments to make errors go away. Fix the code instead, or ask.
- Don't edit generated code (emitted `*.d.ts`, codegen output, `dist/`). Change the source or the generator.

## Style

- Use `interface` for object shapes. Use `type` only for what interfaces can't express, such as unions, intersections, mapped, conditional, and tuple types.
- Don't use `enum`. Use string literal unions or an `as const` object with a derived union:

  ```ts
  export const Status = { Active: "active", Disabled: "disabled" } as const;
  export type Status = (typeof Status)[keyof typeof Status];
  ```

- Use `import type` / `export type` for type-only imports and exports.
- Use `??` and `?.` for defaults and optional access, not `||`, so `0`, `""`, and `false` aren't replaced.
- Use arrow functions for standalone functions, including exported and top-level ones. Exceptions: class methods, generators, overloads, and functions that need hoisting or their own `this`.

  ```ts
  export const parseUser = (input: unknown): User => { /* ... */ };
  ```

## Advanced typing patterns

- Model variants as discriminated unions with a literal `kind`/`type` field instead of optional flags.
- Make `switch` statements over unions exhaustive: ``default: throw new Error(`Unhandled: ${x satisfies never}`)``.
- Use `satisfies` to check a value against a type without widening its inferred type.
- Constrain generics (`<T extends Base>`) and don't add type parameters that appear only once.
- Derive types (`typeof`, `keyof`, indexed access, `Pick`, `Omit`, `ReturnType`, `Awaited`, etc.) instead of re-declaring shapes by hand.

## Errors

- Throw `Error` subclasses with a meaningful `name`. Pass the original error as `cause` when wrapping.
- In `catch`, treat the error as `unknown` and narrow it before using it.
- Don't swallow errors silently.

## Node.js

- Import built-ins with the `node:` prefix: `import { readFile } from "node:fs/promises"`.
- No floating promises. Either `await` them or mark fire-and-forget calls explicitly with `void` and handle their errors.
- Run independent async work concurrently with `Promise.all`, not sequential `await` in a loop.
- Read and validate `process.env` in a single typed config module. The rest of the code imports from that module.

## Testing

- Use the repo's existing test runner and helpers. Never add a different one. If the repo has no test runner, ask before adding one; for plain Node, prefer the built-in `node:test`.
- Name tests by behavior, e.g. `"returns 404 when the user does not exist"`.
- Mock only I/O boundaries (network, filesystem, time, randomness). Don't mock the project's own modules.
- Use fake timers instead of real waits or sleeps.
- Test code follows the same type rules. Use typed mocks (e.g. `vi.mocked`, `jest.mocked`) instead of `as any`.

## Verification

After changing TypeScript files, run the following, using the repo's scripts when they exist (check `package.json` and use the repo's package manager):

1. Format: the repo's formatter (Prettier, Biome, etc.) on the changed files.
2. Typecheck: the repo's `typecheck` script, otherwise `tsc --noEmit` (with `-p` for the relevant project in monorepos).
3. Lint: the repo's lint script, scoped to the changed files when possible.
4. Tests related to the changed code.

Fix any failures your changes introduced. Report pre-existing failures instead of fixing unrelated code.
