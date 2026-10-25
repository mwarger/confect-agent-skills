---
name: confect-test
description: "Generate @confect/test tests for the app's Confect functions (Effect on Convex). Use when the project has a confect/ directory and the user asks to test specs, impls, auth, errors, HTTP routes, or scheduled functions."
---

<!-- GENERATED from convex-agents content/capabilities/test.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Generate Confect tests

Use `@confect/test` and `@effect/vitest` to test Confect functions against an in-memory backend. Test args and returns, auth paths, typed errors, indexes, HTTP routes, and scheduled functions.

Read `references/confect-patterns.md` in the confect skill first. Section E16 has the complete test setup. For platform rules, read the project's `convex/_generated/ai/guidelines.md` if it exists. The Convex guidelines remain the source for platform rules.

## Workflow

1. Install `@confect/test`, `convex-test`, `vitest`, and `@effect/vitest` as dev dependencies. Use the 4.x line of `@effect/vitest` to match Effect v4. `@effect/vitest` needs `vitest` as a peer dependency. `@confect/test` uses `convex-test` inside.
2. Make a test helper file, `test/TestConfect.ts`. In it, export the service and the layer:
   `export const TestConfect = TestConfect_.TestConfect<typeof confectSchema>();`
   `export const layer = TestConfect_.layer(confectSchema, convexSchema, modules);`
   Here `TestConfect_` is the import from `@confect/test`. Section E16 shows the full file. The module glob must match `.ts` and `.js` files in `convex/` and in `convex/_generated/`. Exclude `.d.ts` files. Example: `import.meta.glob(["../convex/**/*.{ts,js}", "!../convex/**/*.d.ts"])`. Codegen writes `convex/_generated` as `.js` plus `.d.ts`. A glob for `.ts` only finds no `_generated` key. Then `convex-test` throws `Could not find the "_generated" directory`.
3. Write tests with `it.effect`. Get the `TestConfect` service. Seed data with `confect.run(...)` and the `DatabaseWriter` service. Call functions with `confect.query`, `confect.mutation`, and `confect.action` and a ref from `refs.public.*` or `refs.internal.*`. Assert on the result.
4. Cover auth with `confect.withIdentity(...)`. Cover error paths with `Effect.flip` or `Effect.catchTag`. Cover HTTP routes with `confect.fetch`. Cover scheduled functions with `finishInProgressScheduledFunctions` and `finishAllScheduledFunctions`.
5. Run vitest. Keep tests deterministic.

```ts
// Bad: a string function name, and no typed error check.
//   const confect = yield* TestConfect.TestConfect;
//   yield* confect.query("notes:listMine" as any);
// Bad: a test that calls a live deployment instead of the in-memory layer.

// Good: one layer per test, a signed-in caller, and a typed error check.
// `./TestConfect` is the helper file from step 2.
import { describe, expect, it } from "@effect/vitest";
import * as Effect from "effect/Effect";

import refs from "../confect/_generated/refs";
import * as TestConfect from "./TestConfect";

const alice = {
  subject: "alice",
  issuer: "https://auth.example.com",
  tokenIdentifier: "https://auth.example.com|alice",
  name: "Alice",
};

describe("notes", () => {
  it.effect("lists notes for the signed-in user", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;
      const asAlice = confect.withIdentity(alice);

      yield* asAlice.mutation(refs.public.users.store);
      yield* asAlice.mutation(refs.public.notes.create, { text: "Hello" });

      const notes = yield* asAlice.query(refs.public.notes.listMine);
      expect(notes.map((note) => note.text)).toEqual(["Hello"]);
    }).pipe(Effect.provide(TestConfect.layer)),
  );

  it.effect("fails with Unauthenticated when nobody is signed in", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;
      const error = yield* confect
        .query(refs.public.notes.listMine)
        .pipe(Effect.flip);
      expect(error._tag).toBe("Unauthenticated");
    }).pipe(Effect.provide(TestConfect.layer)),
  );
});
```

## Setup notes

- Inline `convex-test` in the Vitest config (`server.deps.inline`).
- Exclude `**/*.spec.ts` in the Vitest config. In a Confect project, `*.spec.ts` files are function specs, not tests. Name test files `*.test.ts`.
- Add `/// <reference types="vite/client" />` only to files that call `import.meta.glob`.
- Do not add a `compilerOptions.types` entry for a package that is not installed. TypeScript reports an error for each missing type package. Use the `/// <reference types="vite/client" />` line in the helper file instead.
- `@confect/test` does not register components. `TestConfect.layer` builds the `convex-test` instance inside and does not expose it. If the code under test calls a component, Confect gives no wrapper for this. Use the escape hatch. Build the instance yourself with `convexTest(convexSchema, modules)`. Register the component with `t.registerComponent(...)` or with the component's own test helper, for example `register(t)`. Test those calls with raw `convex-test`, in a separate test file.
- Run `confect codegen` before the tests. The tests import `confect/_generated/*` and `convex/`.

## Rules

- Use `@confect/test` (in-memory), not a live deployment.
- Use refs (`refs.public.*`, `refs.internal.*`). Do not use string function names.
- Cover auth and error paths, not just the happy path.
- Assert typed errors by tag (`Effect.catchTag`, or `_tag` after `Effect.flip`). A function fails with a defect when no `error` schema covers the error. The `error` schema of the function and the error schemas of its middleware both decode as typed errors.
- Keep tests deterministic (no real time or network). For scheduled functions, turn on fake timers with `vi.useFakeTimers()`. For `finishAllScheduledFunctions`, pass the function that advances the fake timers, for example `() => vi.runAllTimers()`.
