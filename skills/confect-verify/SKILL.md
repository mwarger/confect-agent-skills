---
name: confect-verify
description: "Prove a Confect feature works. Use for Confect work (a confect/ directory). Seed, drive as multiple identities with the `withIdentity` method of the TestConfect service, and assert behavior. Include the negative authz cases (wrong user refused, data-scope enforced)."
---

<!-- GENERATED from convex-agents content/capabilities/convex-verify.json — do not edit by hand. -->

# Prove a feature works — seed, drive, assert

A green typecheck proves the code parses; it does not prove a non-owner is actually denied, that a query returns the right rows, or that a mutation has the effect it claims. This capability closes that gap with the loop the whole field is missing: seed → drive → assert, run in-process with `@confect/test` (which wraps `convex-test`) so it needs no deployment. Its highest-value assertions are the NEGATIVE ones — the caller who should be refused — because those are exactly the authz defects the 30-app corpus shows are the #1 real bug and the ones a happy-path demo never catches.

Before you write Confect code or tests, read `confect/references/confect-patterns.md` (the confect skill). The Convex AI guidelines stay the source for platform rules. Example E16 in `confect-patterns.md` holds the full test setup.

## Workflow

1. IDENTIFY the feature to prove: the specific exported query/mutation/action (or a small set) the user just built/changed, and its intended behavior — who should be allowed, what data should come back, what a mutation should change. Find each function through its `FunctionSpec` and its `refs.public.*` or `refs.internal.*` ref. If the intent is unstated, ask one focused question rather than guessing the contract.
2. SET UP `@confect/test`: ensure `@confect/test`, `convex-test`, `@effect/vitest`, and `vitest` are dev deps AND a `vitest.config.ts` sets `test.environment: "edge-runtime"` with `server.deps.inline: ["convex-test"]` — This is the supported setup for `convex-test` in Vitest. Also install `@edge-runtime/vm`. Exclude `**/*.spec.ts` in the Vitest config, because Vitest treats `*.spec.ts` as test files and Confect function specs use that name. Run `npx confect codegen` first, so `confect/_generated/` and `convex/` exist. Then make one local module, `test/TestConfect.ts` (see E16 in `confect-patterns.md`). In it, export the service tag `TestConfect = TestConfect_.TestConfect<typeof confectSchema>()`. Also export `layer = TestConfect_.layer(confectSchema, convexSchema, import.meta.glob(["../convex/**/*.ts", "!../convex/**/*.d.ts"]))`. `TestConfect_.layer` is a function with three arguments. Do not pass the uncalled function to `Effect.provide`. In each test, run `Effect.provide(layer)` with the exported `layer`. Each test gets a clean database. Reuse the project's existing test setup if present (compose with the `confect-test` capability, don't fork it).
3. SEED realistic data through the app's OWN functions where possible (so the seed exercises the same Schema decoding and mutations a real user would), falling back to `confect.run(Effect.gen(...), returnsSchema)` with the `DatabaseWriter` service (`writer.table("notes").insert(...)`) for fixtures the public API can't create. Seed at least: the caller's own rows AND a second user's rows, so cross-user access is testable.
4. DRIVE the feature as DIFFERENT identities with `confect.withIdentity({ subject, issuer, tokenIdentifier, ... })`: call the function as (a) the legitimate owner, (b) a different authenticated user, and (c) unauthenticated (the `confect` service with no identity). Get the service with `const confect = yield* TestConfect.TestConfect`. Call each function with `asUser.query(refs.public.x.y, args)`, `asUser.mutation(...)`, or `asUser.action(...)`. Use the real identity shape the app's auth uses (subject/tokenIdentifier), matching how ownership is resolved.
5. ASSERT behavior — POSITIVE and NEGATIVE. Each call returns an `Effect`. Run it with `yield*` inside `it.effect`:
   - positive: the owner gets the expected rows / the mutation made the expected change (`expect(yield* asOwner.query(refs.public.x.y, args)).toEqual(...)`).
   - NEGATIVE (the load-bearing half): a different user calling the same function is REFUSED. A refusal is a typed error from the function spec's `error` Schema. The tag must match the error that the spec declares. If the code hides other users' documents (see `confect-authz`), a missing document and another user's document fail with the same not-found error. Use `Effect.flip` and check the tag: `const error = yield* asOther.mutation(refs.public.notes.remove, { noteId }).pipe(Effect.flip); expect(error._tag).toBe("NoteNotFound")`. Use `Effect.catchTag("NoteNotFound", ...)` for typed recovery. An unauthenticated caller is refused where auth is required (for example `Unauthenticated`). A feature is not proven until the wrong caller is shown to be blocked.
   - If the function dies (a defect, for example after `Effect.orDie`) instead of failing with a typed error, `Effect.flip` does not see it. Use `Effect.exit` and assert on the failed `Exit`. Better: declare the refusal as a typed `error` in the spec, so the test and the client can both name it.
   - HTTP routes: a refusal is a status code, not a typed error. Call `confect.fetch("/path", init)` and assert on `res.status` (for example 401 or 403).
   - Plain Convex functions (`FunctionSpec.convex*`) have no typed error. `Ref.runWithCodec` surfaces their refusals as defects. Use `Effect.exit` for them.
   - data-scope: a list/query returns ONLY the caller's rows, never the second user's (assert the second user's row is absent).
6. RUN the tests (`npx vitest run`) and report: what was proven (each positive + negative assertion that passed), and — critically — any assertion that FAILED, because a failed negative assertion is a real authz hole found before ship. Emit findings on the bus (specs/finding.schema.json, class authz/correctness, evidence kind probe-result with the exact failing call) for anything that didn't behave. Also run `tsc --noEmit`. Codegen success alone is not a type check.
7. Do NOT weaken a test to make it pass: if the owner-only query returns another user's row, the FIX is in the function (hand to confect-authz), not in the assertion. A test changed until it's green proves nothing.

## Rules

- Prove behavior, not compilation: every verification includes at least one NEGATIVE assertion (a caller who should be refused is refused) — the happy path alone is not proof.
- Drive the feature as multiple identities with `confect.withIdentity` (owner, other user, unauthenticated) using the app's real subject/tokenIdentifier shape.
- Seed both the caller's rows AND a second user's rows so cross-user access and data-scope are actually testable.
- Use a vitest.config.ts with environment 'edge-runtime' and convex-test inlined; author it, don't just author the test file. Add `/// <reference types="vite/client" />` only to the file that calls `import.meta.glob`.
- Assert refusals as typed errors (`Effect.flip` and `_tag`, or `Effect.catchTag`). Do not match error message text.
- Run in-process with `@confect/test` — no deployment needed; compose with the `confect-test` capability's setup rather than forking it.
- Never weaken an assertion to make it pass: a failing negative test is a real defect → hand the fix to confect-authz/confect-expert, don't edit the test until it's green.
- Emit a bus finding for any assertion that failed (authz/correctness, evidence: the failing probe call) so a composite pass or self-heal can pick it up.
- `@confect/test` gives no way to register a component. `TestConfect.layer` calls `convexTest` inside and does not expose the `TestConvex` handle. For code that calls a component, write a separate raw `convex-test` file that calls `convexTest(...).registerComponent(...)`. Or do not call that code path in the Confect test.
- This drives a SPECIFIC built feature; a request to set up a test framework generally is the `confect-test` capability.
