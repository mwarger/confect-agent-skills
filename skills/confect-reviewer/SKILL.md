---
name: confect-reviewer
description: "Confect code reviewer — security, auth, schemas, performance, and pattern checks for code in a confect/ directory (Effect on Convex). Use to review or audit Confect specs, impls, tables, and middleware before shipping."
---

<!-- ADAPTED BY HAND for Confect from convex-agents content/capabilities/convex-reviewer.json. Edit this file directly. -->

# Confect Code Reviewer

Structured review of Confect code for security, authorization, schemas, performance, and table design. Applies a Confect-specific checklist and flags anti-patterns with severity (Critical / Important / Suggestion).

Confect changes how code is written. It does not change how Convex behaves. Every Convex limit and rule still applies. For the Confect form of each Convex concept, read the confect skill's `references/confect-patterns.md`. For the platform rules that still apply (limits, indexes, OCC, scheduling), read the section "Platform rules that still apply" in the same file.

## Workflow

1. First pass — Security: verify every public function reads the identity through `Auth` (`auth.getUserIdentity`) or through middleware that provides the user. Verify resource ownership before reads/writes. Confirm no client-provided user IDs are trusted. Confirm scheduled functions target `refs.internal.*` not `refs.public.*`.
2. Second pass — Performance: confirm every read names an index (`.index(...)`) and no `.stream()` plus `Stream.filter`, and no `filter` argument of `paginate`, stands in for an index range. Verify all foreign-key fields have indexes. Confirm no `Date.now()` or `Clock.currentTimeMillis` in query impls. Confirm `.collect()` is not used on unbounded queries (use `take(n)` or `publicPaginatedQuery`). In escape hatch code (plain Convex functions and raw ctx), confirm no `ctx.db.query(...).filter(...)`.
3. Third pass — Code quality: confirm `args` and `returns` schemas on every public function spec (a paginated spec takes `item` instead of `returns`, and a plain `convex*` spec uses Convex validators), no TypeScript `any` (`as any` or an `any` parameter), no `Schema.Any` or `Schema.Unknown`, no `Schema.Number` (use `Schema.Finite`), Effects are yielded (an Effect that is not yielded never runs), arrays in documents are bounded (<8192 elements).
4. Fourth pass — Confect structure: confirm specs do not value-import `@confect/server`. Confirm each expected failure has an `error` schema and uses `return yield* new MyError(...)`. Confirm other failures use `Effect.orDie`. Confirm `GetByIdFailure` and `GetByIndexFailure` map to a declared error, not to `null`. Confirm each group impl layer provides every `FunctionImpl` and middleware impl. Confirm no hand edits in `confect/_generated/` or the generated files in `convex/`.
5. Fifth pass — Escape hatches: for each plain Convex function in the spec tree (`FunctionSpec.convexPublicQuery` and the other `convex*` constructors) and each use of the raw `QueryCtx`, `MutationCtx`, or `ActionCtx`, apply the Convex rules by hand, because Confect does not check them. Confirm the function has `args` and `returns` Convex validators. Confirm the identity check (`ctx.auth.getUserIdentity()`). Confirm reads use an index and not `.filter()`. Confirm scheduling targets `internal.*` and not `api.*`. Confirm every promise is awaited.
6. Report findings grouped by severity; explain why each issue matters and suggest a fix.

## Rules

- Flag missing auth checks as Critical — any unauthenticated public mutation is a data-loss risk. A group without an auth middleware needs an `Auth` check in each function.
- Flag trust in a client-provided user ID as Critical. Look the user up by `tokenIdentifier`.
- Flag reads without an index range, or a post-scan predicate, as Important — it is a full table scan. Confect queries have no `.filter()`, but `paginate(opts, filter)` accepts one and it does not cut the rows read. Raw ctx code can call `.filter()` too. A `.stream()` with `Stream.filter` does not reduce the rows read.
- Flag `Date.now()` or `Clock.currentTimeMillis` in query impls as Important — Convex evicts the query from the cache, so it breaks reactivity. Pass the time as an argument. Confect 10.0.0 does not stub `Date.now()`.
- Flag missing `args` schemas as Important. An omitted `args` means "no arguments". A missing `returns` on a Confect spec does not compile, so it is not a finding. Do not flag `item` on a paginated spec.
- Flag raw Convex code in an escape hatch that breaks a rule above (auth check, index use, `api.*` scheduling, promise not awaited) with the same severity as the rule.
- Flag TypeScript `any` as Suggestion. Use a schema type or `unknown` with a narrow.
- Flag scheduling to `refs.public.*` (not `refs.internal.*`) as Important.
- Flag a function with no `error` schema that fails with an expected error as Important. Without it, every failure is a defect and the caller cannot tell "not found" from a crash.
- Flag `Schema.Number` as Suggestion. Use `Schema.Finite`. `Schema.Number` accepts `NaN` and `Infinity`.
- Flag `Effect.fail(new MyError(...))` as Suggestion. Use `return yield* new MyError(...)`.
- Flag an unbounded `.collect()` or an unbounded array field as Important.
- Always explain why a change is needed, not just what to change.

Example finding:

```ts
// Bad: reads up to 1000 notes, then filters. Matches after the first 1000 are missed.
const notes = yield* reader.table("notes").index("by_creation_time").take(1000);
const minePostFilter = notes.filter((note) => note.authorId === user._id);
```

```ts
// Good: the index range holds the predicate.
const mine = yield* reader
  .table("notes")
  .index("by_authorId", (q) => q.eq("authorId", user._id), "desc")
  .take(50);
```
