# Confect patterns

This file maps each Convex concept to its Confect equivalent. Confect is
Effect on Convex. Confect changes how you write code. It does not change how
the Convex platform behaves. Every Convex limit and rule still applies.

Use this file when you write or review code in a project that has a
`confect/` directory.

## Scope and sources

- Versions: `@confect/core`, `@confect/server`, `@confect/cli`, and
  `@confect/test` 10.0.0, with `effect` 4.0.0 and `convex` 1.46.
- Source of truth: the package sources (`src/` and `dist/*.d.ts`). The docs at
  https://confect.dev can describe a newer version. When the docs and the
  installed source differ, the source wins. See
  [Version notes](#version-notes).
- Every code block in this file was compiled with `tsc --noEmit` (strict,
  `exactOptionalPropertyTypes`) after `confect codegen`. The test files also
  ran with Vitest. There are 54 compiled code blocks.
- All examples use neutral domains: users, notes, messages, tasks, files,
  documents.

## Project layout

```
confect/
  tables/<name>.ts        one table per file; the file name is the table name
  <group>.spec.ts         function specs (client-safe)
  <group>.impl.ts         function implementations (server only)
  <group>.ts              optional plain Convex functions for the group
  middleware/<Name>.spec.ts and <Name>.impl.ts
  http.ts                 optional HTTP router
  crons.ts                optional cron jobs
  auth.ts                 optional auth config
  _generated/             written by `confect codegen`; do not edit
convex/
  convex.config.ts        you own this file (components)
  tsconfig.json           you own this file
  everything else         written by `confect codegen`; do not edit
```

Rules:

- Keep `confect/` next to `convex/`.
- Do not edit `confect/_generated/` or the generated files in `convex/`.
- A spec is bundled into the client. Nothing a `*.spec.ts` file reaches can
  value-import `@confect/server`. Use `import type` for server types.
  `confect codegen` enforces this rule.
- The file path is the API path. `confect/notes.spec.ts` gives
  `refs.public.notes.*` and the Convex function name `notes:<name>`.
  `confect/billing/invoices.spec.ts` gives the group `billing.invoices`.
  A rename changes the API.
- Vitest treats `*.spec.ts` as test files by default. Exclude
  `**/*.spec.ts` in the Vitest config.

Workflow:

1. `npx convex dev --once` creates the deployment and `convex/` (first time).
2. `npx confect codegen` writes `confect/_generated/` and `convex/`.
3. During development, run `npx confect dev` and `npx convex dev` in two
   terminals.
4. Gate on `tsc --noEmit`. Codegen success alone is not enough.

Deploy, environment variables (`npx convex env set`), the dashboard, logs,
insights, and the Convex MCP tools work as in plain Convex. They see the
generated `convex/` modules, so function names follow the group paths.

## Concept map

"Ctx" names the function types that can use the service: Q = query,
M = mutation, A = action (Convex and Node runtimes), H = HTTP route.
The services come from the generated module `confect/_generated/services`.

| Convex concept | Confect equivalent | Export and module | Example |
| --- | --- | --- | --- |
| `convex/schema.ts` with `defineSchema` | One file per table in `confect/tables/`. Default-export a `Table`. Codegen writes `convex/schema.ts`. | `Table.make` (`@confect/core/Table`) | [E1](#e1-tables-and-indexes) |
| `defineTable(v.object({...}))` | `Table.make(() => Schema.Struct({...}))` | `Table.make` (`@confect/core/Table`) | [E1](#e1-tables-and-indexes) |
| Union table `defineTable(v.union(...))` | `Table.make(() => Schema.Union([...]))` | `Table.make` (`@confect/core/Table`) | [E1](#e1-tables-and-indexes) |
| `.index("by_a_and_b", ["a", "b"])` | Same call on the table builder | `UnnamedTable.index` (`@confect/core/Table`) | [E1](#e1-tables-and-indexes) |
| `.index(name, { fields, staged: true })` | Not supported | None | [Gaps](#gaps) |
| `.searchIndex(name, {...})` | Same call on the table builder | `UnnamedTable.searchIndex` (`@confect/core/Table`) | [E1](#e1-tables-and-indexes) |
| `.vectorIndex(name, {...})` | Same call on the table builder | `UnnamedTable.vectorIndex` (`@confect/core/Table`) | [E1](#e1-tables-and-indexes) |
| `v.*` validators | Effect `Schema` values. See [Validators](#validators-to-effect-schema). | `effect/Schema` | [E1](#e1-tables-and-indexes) |
| `v.id("notes")` | `Id("notes")` for your tables | `Id` (`confect/_generated/id`) | [E1](#e1-tables-and-indexes) |
| `v.id("_storage")`, `v.id("_scheduled_functions")` | `GenericId.GenericId("_storage")` | `GenericId.GenericId` (`@confect/core/GenericId`) | [E1](#e1-tables-and-indexes) |
| `Id<"notes">` type | `GenericId.GenericId<"notes">` | `GenericId.GenericId` type (`@confect/core/GenericId`) | [E3](#e3-reads) |
| `Doc<"notes">` type | `NotesDoc`, or `typeof notes.Doc.Type` | `NotesDoc` (`confect/_generated/docs`) | [E2](#e2-function-specs-and-implementations) |
| `schema.doc("notes")`, `docValidator` | `notes.Doc` (fields plus `_id` and `_creationTime`) | default export of `confect/_generated/tables/notes` | [E2](#e2-function-specs-and-implementations) |
| Reuse object validator fields (`.fields`, `.pick`) | `notes.Fields.fields.<name>` | `Table.Fields` (`@confect/core/Table`) | [E2](#e2-function-specs-and-implementations) |
| `query`, `mutation`, `action` | `FunctionSpec.publicQuery`, `publicMutation`, `publicAction` in the spec | `@confect/core/FunctionSpec` | [E2](#e2-function-specs-and-implementations) |
| `internalQuery`, `internalMutation`, `internalAction` | `FunctionSpec.internalQuery`, `internalMutation`, `internalAction` | `@confect/core/FunctionSpec` | [E2](#e2-function-specs-and-implementations) |
| Function file (`convex/notes.ts`) | A `GroupSpec` default-exported from `notes.spec.ts` | `GroupSpec.make` (`@confect/core/GroupSpec`) | [E2](#e2-function-specs-and-implementations) |
| `handler: async (ctx, args) => ...` | `FunctionImpl.make(databaseSchema, group, "name", (args) => Effect)` | `FunctionImpl.make` (`@confect/server/FunctionImpl`) | [E2](#e2-function-specs-and-implementations) |
| All functions of a file registered | `GroupImpl.make(...)`, then `GroupImpl.finalize`. Finalize does not typecheck until every spec function has an impl. | `GroupImpl.make`, `GroupImpl.finalize` (`@confect/server/GroupImpl`) | [E2](#e2-function-specs-and-implementations) |
| `args: {...}` | `args: () => ({ field: Schema })`. A field map, not a `Schema.Struct`. Omit it for no args. | `FunctionSpec` options | [E2](#e2-function-specs-and-implementations) |
| `returns: v.null()` and no return value | `returns: () => Schema.Null`. `Schema.Undefined` and `Schema.Void` are rejected. | `effect/Schema` | [E2](#e2-function-specs-and-implementations) |
| `ctx.db` reads | `DatabaseReader` service (Q, M) | `DatabaseReader` (`confect/_generated/services`) | [E3](#e3-reads) |
| `ctx.db.get("notes", id)` | `reader.table("notes").get(id)`. It fails with `GetByIdFailure`. It does not return `null`. | `GetByIdFailure` (`@confect/server/QueryInitializer`) | [E3](#e3-reads) |
| `.withIndex(name, (q) => ...)` | `reader.table(t).index(name, (q) => ..., order?)` | `QueryInitializer.index` (`@confect/server/QueryInitializer`) | [E3](#e3-reads) |
| `.order("desc")` | Last argument of `.index(...)` | `QueryInitializer.index` | [E3](#e3-reads) |
| Table scan (`ctx.db.query("t")`) | `.index("by_creation_time")`. Every query names an index. | `QueryInitializer.index` | [E3](#e3-reads) |
| `.take(n)`, `.first()`, `.collect()` | Same names. `first()` gives an `Option`. | `OrderedQuery` (`@confect/server/OrderedQuery`) | [E3](#e3-reads) |
| `.unique()` | `reader.table(t).get(indexName, ...values)`. It fails with `GetByIndexFailure` when there is no match. More than one match is a defect. | `GetByIndexFailure` (`@confect/server/QueryInitializer`) | [E5](#e5-auth-and-middleware) |
| `.filter(...)` | Not on queries. See [Gaps](#gaps). | None | [Gaps](#gaps) |
| `for await (const row of query)` | `.stream()` gives an Effect `Stream` | `OrderedQuery.stream` | [Gaps](#gaps) |
| `.withSearchIndex(name, (q) => ...)` | `reader.table(t).search(name, (q) => ...)` | `QueryInitializer.search` | [E3](#e3-reads) |
| `paginationOptsValidator` and `.paginate(opts)` | `FunctionSpec.publicPaginatedQuery({ item })`. Confect adds the `paginationOpts` arg and the result schema. Pass `paginationOpts` unchanged to `.paginate`. | `FunctionSpec.publicPaginatedQuery`, `internalPaginatedQuery` (`@confect/core/FunctionSpec`); `PaginationOptions.PaginationOptions`, `PaginationResult.PaginationResult` (`@confect/core`) | [E3](#e3-reads) |
| `ctx.db.insert`, `patch`, `replace`, `delete` | `writer.table(t).insert`, `patch`, `replace`, `delete` (M) | `DatabaseWriter` (`confect/_generated/services`) | [E4](#e4-writes) |
| `ctx.db.system.get("_storage", id)` | `reader.table("_storage").get(id)` | `DatabaseReader` | [E8](#e8-file-storage) |
| `ctx.db.normalizeId` | Not wrapped. Use the raw ctx. | `QueryCtx`, `MutationCtx` (`confect/_generated/services`) | [E12](#e12-plain-convex-functions-and-escape-hatches) |
| `ctx.auth.getUserIdentity()` | `auth.getUserIdentity` (an Effect, not a function). It fails with `NoUserIdentityFoundError`. (Q, M, A, H) | `Auth`, `NoUserIdentityFoundError` (`@confect/server/Auth`) | [E5](#e5-auth-and-middleware) |
| `convex/auth.config.ts` | `confect/auth.ts`. Codegen copies it to `convex/auth.config.ts`. | None | [E14](#e14-auth-config) |
| Custom functions (`convex-helpers`) | Middleware: a spec plus an impl, attached to a group or one function | `MiddlewareSpec.MiddlewareSpec` (`@confect/core/MiddlewareSpec`); `MiddlewareImpl.provides`, `make`, `makeByFunctionType` (`@confect/server/MiddlewareImpl`) | [E5](#e5-auth-and-middleware) |
| `api.notes.list`, `internal.notes.list` | `refs.public.notes.list`, `refs.internal.notes.list` | default export of `confect/_generated/refs` | [E6](#e6-calling-other-functions) |
| `ctx.runQuery` | `runQuery(ref, args?, options?)` (Q, M, A, H) | `QueryRunner` (`confect/_generated/services`) | [E6](#e6-calling-other-functions) |
| `ctx.runMutation` | `runMutation(ref, args?, options?)` (M, A, H) | `MutationRunner` | [E6](#e6-calling-other-functions) |
| `ctx.runAction` | `runAction(ref, args?)` (A, H) | `ActionRunner` | [E6](#e6-calling-other-functions) |
| `transactionLimits` on nested calls | Third argument: `{ transactionLimits }` (Q, M only) | `QueryRunner`, `MutationRunner` | [E6](#e6-calling-other-functions) |
| `ctx.scheduler.runAfter`, `runAt`, `cancel` | `runAfter(Duration, ref, args?)`, `runAt(DateTime, ref, args?)`, `cancel(id)` (M, A, H) | `Scheduler` (`confect/_generated/services`) | [E7](#e7-scheduling-and-batches) |
| `ctx.meta.getTransactionMetrics()` | `getMetrics()` (Q, M) | `TransactionMetadata` | [E7](#e7-scheduling-and-batches) |
| Function and request metadata | `getFunction()`, `getDeployment()` (Q, M, A, H); `get()` (M, A, H) | `ExecutionMetadata`, `RequestMetadata` | None |
| `ctx.storage.*` | One service: `getUrl` (Q, M, A, H), `generateUploadUrl` and `delete` (M, A, H), `get` and `store` (A, H). Missing blobs fail with `BlobNotFoundError`. | `Storage` (`confect/_generated/services`); `BlobNotFoundError` (`@confect/server/BlobNotFoundError`) | [E8](#e8-file-storage) |
| `ctx.vectorSearch` | `vectorSearch(table, index, query)` (A only) | `VectorSearch` (`confect/_generated/services`) | [E9](#e9-vector-search) |
| `process.env`, `env` from `_generated/server` | `Config.String("NAME")` and other `Config` readers | `effect/Config`; `ConvexConfigProvider.make` (`@confect/server/ConvexConfigProvider`) | [E10](#e10-environment-variables) |
| `"use node"` action file | A Node group: `GroupSpec.makeNode()` with `FunctionSpec.publicNodeAction` or `internalNodeAction` | `@confect/core/GroupSpec`, `@confect/core/FunctionSpec` | [E11](#e11-node-actions) |
| `ConvexError` and error handling | `error: () => MyError` in the spec. Fail with `return yield* new MyError(...)`. Callers get the decoded error. | `Schema.TaggedError` (`effect/Schema`) | [E2](#e2-function-specs-and-implementations) |
| Components in `convex/convex.config.ts` | No change. You own that file. | None | [E6](#e6-calling-other-functions) |
| `components` from `_generated/api` | `components` registry from Confect codegen | `components` (`confect/_generated/components`) | [E6](#e6-calling-other-functions) |
| Component client calls (`limiter.limit(ctx, ...)`) | Pass the raw ctx service to the client | `MutationCtx`, `QueryCtx`, `ActionCtx` (`confect/_generated/services`) | [E6](#e6-calling-other-functions) |
| App functions that a component calls | Plain Convex functions in the spec tree | `FunctionSpec.convexInternalMutation` and the other `convex*` constructors (`@confect/core/FunctionSpec`) | [E12](#e12-plain-convex-functions-and-escape-hatches) |
| `createFunctionHandle` | Not wrapped. Use `convex/server` with the ref's function reference. | `Ref.getFunctionReference` (`@confect/core/Ref`) | [E12](#e12-plain-convex-functions-and-escape-hatches) |
| `QueryCtx`, `MutationCtx`, `ActionCtx` types | Raw ctx services (escape hatch only) | `QueryCtx`, `MutationCtx`, `ActionCtx` (`confect/_generated/services`) | [E12](#e12-plain-convex-functions-and-escape-hatches) |
| `convex/http.ts` with `httpRouter` and `httpAction` | `confect/http.ts` default-exports `HttpRouter.make(layer)`. Define routes with `effect/http-api` and `effect/http`. | `HttpRouter.make` (`@confect/server/HttpRouter`) | [E13](#e13-http-actions) |
| A plain `httpAction` route | `http.route({ ..., handler: httpActionGeneric(...) })` on the returned router | `httpActionGeneric` (`convex/server`) | [E13](#e13-http-actions) |
| `convex/crons.ts` with `cronJobs()` | `confect/crons.ts` default-exports `CronJobs.make().add(CronJob.make(...))` | `CronJobs.make` (`@confect/server/CronJobs`); `CronJob.make` (`@confect/server/CronJob`) | [E15](#e15-cron-jobs) |
| `crons.cron(...)`, `crons.interval(...)` | A `Cron` schedule, or a `Duration` interval | `effect/Cron`, `effect/Duration` | [E15](#e15-cron-jobs) |
| `console.log` | `Effect.log`, `Effect.logWarning`, `Effect.logError` | `effect/Effect` | None |
| `npx convex dev` codegen | `npx confect codegen` or `npx confect dev`, plus `npx convex dev` | `@confect/cli` | [Project layout](#project-layout) |
| `convex-test` with `convexTest(schema, modules)` | `TestConfect.layer(schema, convexSchema, modules)` and the `TestConfect` service | `TestConfect.layer`, `TestConfect.TestConfect` (`@confect/test/TestConfect`) | [E16](#e16-testing) |
| `t.withIdentity`, `t.run`, `t.fetch`, scheduled-function helpers | `withIdentity`, `run`, `fetch`, `finishInProgressScheduledFunctions`, `finishAllScheduledFunctions` | `TestConfect` service | [E16](#e16-testing) |

## Validators to Effect Schema

Codegen compiles each Effect schema to a Convex validator. This table shows
the validator that each schema compiled to in the check project. The
`Encoded` side of a schema must be a Convex value. The `Type` side can be any
TypeScript type.

| Convex validator | Effect schema | Notes |
| --- | --- | --- |
| `v.string()` | `Schema.String` | Checks such as `.check(Schema.isMaxLength(2000))` run in Confect, not in the Convex validator. |
| `v.float64()`, `v.number()` | `Schema.Finite` | Prefer `Schema.Finite`. `Schema.Number` also compiles to `v.float64()`, but it accepts `NaN` and `Infinity`. |
| `v.int64()` | `Schema.BigInt` | Range is -2^63 to 2^63-1. |
| `v.boolean()` | `Schema.Boolean` | |
| `v.null()` | `Schema.Null` | Use it for "returns nothing". |
| `v.bytes()` | `Schema.instanceOf(ArrayBuffer)` | |
| `v.id("notes")` | `Id("notes")` | Use `GenericId.GenericId("_storage")` for system tables. |
| `v.array(x)` | `Schema.Array(x)` | |
| `v.object({...})` | `Schema.Struct({...})` | |
| `v.record(k, x)` | `Schema.Record(k, x)` | |
| `v.union(a, b)` | `Schema.Union([a, b])` | |
| `v.literal("a")` | `Schema.Literal("a")` | |
| `v.union(v.literal("a"), v.literal("b"))` | `Schema.Literals(["a", "b"])` | |
| `v.optional(x)` | `Schema.optionalKey(x)` | `Schema.optional(x)` also compiles to `v.optional`. With `exactOptionalPropertyTypes`, prefer `optionalKey`. |
| `v.union(x, v.null())` | `Schema.NullOr(x)` | |
| `v.string()` (wire) to `URL` (code) | `Schema.URLFromString` | A transformation. The wire value is a string. |
| `v.union(x, v.null())` (wire) to `Option` (code) | `Schema.OptionFromNullOr(x)` | |
| `v.float64()` (wire) to `DateTime` (code) | `Schema.DateTimeUtcFromMillis` | |
| `v.any()` | `Schema.Unknown`, `Schema.Any`, or a recursive schema | Avoid. |
| None | `Schema.Undefined`, `Schema.Void`, `Schema.Symbol`, template literals | Rejected by codegen. |

Function `args` is a field map (`() => ({ a: Schema.String })`). You cannot
pass a `Schema.Struct`, and you cannot attach a top-level check to `args`. For
a cross-field check, put the checked struct under one argument field.

## Examples

Each example is a complete file, or an excerpt of a complete file in the same
project. An excerpt uses the imports shown in the "imports" block for its
file.

### E1. Tables and indexes

Name every index after all of its fields, in order.

`confect/tables/users.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

export default Table.make(() =>
  Schema.Struct({
    tokenIdentifier: Schema.String,
    name: Schema.String,
    email: Schema.optionalKey(Schema.String),
  }),
).index("by_tokenIdentifier", ["tokenIdentifier"]);
```

`confect/tables/notes.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "../_generated/id";

export default Table.make(() =>
  Schema.Struct({
    authorId: Id("users"),
    text: Schema.String.check(Schema.isMaxLength(2000)),
    tag: Schema.optionalKey(Schema.String),
  }),
)
  .index("by_authorId", ["authorId"])
  .index("by_authorId_and_tag", ["authorId", "tag"]);
```

A search index:

`confect/tables/messages.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

export default Table.make(() =>
  Schema.Struct({
    channel: Schema.String,
    author: Schema.String,
    body: Schema.String,
  }),
)
  .index("by_channel", ["channel"])
  .searchIndex("search_body", {
    searchField: "body",
    filterFields: ["channel"],
  });
```

A vector index. `dimensions` must equal the length of the stored vectors:

`confect/tables/documents.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

export default Table.make(() =>
  Schema.Struct({
    title: Schema.String,
    category: Schema.String,
    embedding: Schema.Array(Schema.Finite),
  }),
).vectorIndex("by_embedding", {
  vectorField: "embedding",
  dimensions: 1536,
  filterFields: ["category"],
});
```

A discriminated union table:

`confect/tables/results.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

// A discriminated union table. Each member is a Schema.Struct.
export default Table.make(() =>
  Schema.Union([
    Schema.Struct({
      kind: Schema.Literal("error"),
      errorMessage: Schema.String,
    }),
    Schema.Struct({
      kind: Schema.Literal("success"),
      value: Schema.Finite,
    }),
  ]),
);
```

One field for each Convex value type:

`confect/tables/counters.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

// One field per Convex value type, to show the validator each schema compiles to.
export default Table.make(() =>
  Schema.Struct({
    name: Schema.String, // v.string()
    total: Schema.BigInt, // v.int64()
    ratio: Schema.Finite, // v.float64()
    isActive: Schema.Boolean, // v.boolean()
    lastValue: Schema.NullOr(Schema.Finite), // v.union(v.float64(), v.null())
    checksum: Schema.optionalKey(Schema.instanceOf(ArrayBuffer)), // v.optional(v.bytes())
    labels: Schema.Record(Schema.String, Schema.String), // v.record(v.string(), v.string())
    samples: Schema.Array(Schema.Finite), // v.array(v.float64())
    unit: Schema.Literals(["ms", "count"]), // v.union(v.literal("ms"), v.literal("count"))
  }),
);
```

A reference to a system table. The generated `Id` in 10.0.0 accepts only your
own tables:

`confect/tables/files.ts`

```ts
import { GenericId, Table } from "@confect/core";
import * as Schema from "effect/Schema";

export default Table.make(() =>
  Schema.Struct({
    name: Schema.String,
    storageId: GenericId.GenericId("_storage"),
  }),
);
```

Other tables used by the examples:

`confect/tables/tasks.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

export default Table.make(() =>
  Schema.Struct({
    text: Schema.String,
    isCompleted: Schema.Boolean,
    dueAt: Schema.Finite,
  }),
)
  .index("by_isCompleted", ["isCompleted"])
  .index("by_isCompleted_and_dueAt", ["isCompleted", "dueAt"]);
```

### E2. Function specs and implementations

The spec declares names, args, returns, and errors. It is client-safe.
Errors are `Schema.TaggedError` classes. Export them from the spec so that
the impl and the callers can use them.

`confect/notes.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "./_generated/id";
import notes from "./_generated/tables/notes";
import RequireUser from "./middleware/RequireUser.spec";

export class NoteNotFound extends Schema.TaggedError<NoteNotFound>()(
  "NoteNotFound",
  { noteId: Id("notes") },
) {}

// Derive arg fields from the table instead of repeating them.
const editableFields = notes.Fields.fields;

export default GroupSpec.make()
  .middleware(RequireUser)
  .addFunction(
    FunctionSpec.publicQuery({
      name: "get",
      args: () => ({ noteId: Id("notes") }),
      returns: () => notes.Doc,
      error: () => NoteNotFound,
    }),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "listMine",
      returns: () => Schema.Array(notes.Doc),
    }),
  )
  .addFunction(
    FunctionSpec.publicPaginatedQuery({
      name: "listMineByTag",
      args: () => ({ tag: Schema.String }),
      item: () => notes.Doc,
    }),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "create",
      args: () => ({ text: editableFields.text, tag: editableFields.tag }),
      returns: () => Id("notes"),
    }),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "updateText",
      args: () => ({ noteId: Id("notes"), text: editableFields.text }),
      returns: () => Schema.Null,
      error: () => NoteNotFound,
    }),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "remove",
      args: () => ({ noteId: Id("notes") }),
      returns: () => Schema.Null,
      error: () => NoteNotFound,
    }),
  );
```

The impl provides one `FunctionImpl` for each spec function, plus the
middleware impls. Merge them into one layer:

`confect/notes.impl.ts` (imports for the excerpts below)

```ts
import type { GenericId } from "@confect/core";
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import databaseSchema from "./_generated/schema";
import { DatabaseReader, DatabaseWriter } from "./_generated/services";
import { CurrentUser } from "./middleware/RequireUser.spec";
import RequireUserImpl from "./middleware/RequireUser.impl";
import notes, { NoteNotFound } from "./notes.spec";
```

`confect/notes.impl.ts` (excerpt)

```ts
export default GroupImpl.make(databaseSchema, notes).pipe(
  Layer.provide(
    Layer.mergeAll(
      get,
      listMine,
      listMineByTag,
      create,
      updateText,
      remove,
      RequireUserImpl,
    ),
  ),
  GroupImpl.finalize,
);
```

Typed errors:

- A function with an `error` schema sends its typed failures to the caller as
  a Convex `ConvexError`. The encoded error is in `ConvexError.data`. The
  Confect clients and `@confect/test` decode it.
- A mutation that fails rolls back all its writes, as in plain Convex.
- A function without an `error` schema turns every failure into a defect.
  Clear expected failures with `Effect.orDie` or map them to a declared error.
- A defect is not sent as a typed error. The caller sees a generic failure.

`confect/errorStyles.ts` (excerpt)

```ts
// Bad: no error schema, so the caller cannot tell "not found" from a crash.
export const getWithoutError = FunctionSpec.publicQuery({
  name: "get",
  args: () => ({ noteId: Id("notes") }),
  returns: () => Schema.NullOr(notes.Doc),
});
```

`confect/errorStyles.ts` (excerpt)

```ts
// Good: declare the failure, and fail with it in the impl.
export const getWithError = FunctionSpec.publicQuery({
  name: "get",
  args: () => ({ noteId: Id("notes") }),
  returns: () => notes.Doc,
  error: () => NoteNotFound,
});
```

### E3. Reads

`get(id)` fails with `GetByIdFailure` when the document does not exist. Map it
to your own error. A missing document and a document that belongs to another
user must fail the same way:

`confect/notes.impl.ts` (excerpt)

```ts
const getOwnedNote = (noteId: GenericId.GenericId<"notes">) =>
  Effect.gen(function* () {
    const { user } = yield* CurrentUser;
    const reader = yield* DatabaseReader;

    const note = yield* reader
      .table("notes")
      .get(noteId)
      .pipe(
        Effect.catchTags({
          GetByIdFailure: () => Effect.fail(new NoteNotFound({ noteId })),
          DocumentDecodeError: Effect.die,
        }),
      );

    if (note.authorId !== user._id) {
      return yield* new NoteNotFound({ noteId });
    }

    return note;
  });
```

Use an index, and keep the result bounded with `take(n)` or `paginate`:

`confect/notes.impl.ts` (excerpt)

```ts
const listMine = FunctionImpl.make(databaseSchema, notes, "listMine", () =>
  Effect.gen(function* () {
    const { user } = yield* CurrentUser;
    const reader = yield* DatabaseReader;

    // Bounded: take(n), not collect(), on a table that can grow.
    return yield* reader
      .table("notes")
      .index("by_authorId", (q) => q.eq("authorId", user._id), "desc")
      .take(50);
  }).pipe(Effect.orDie),
);
```

Pagination. Pass `paginationOpts` to `.paginate` unchanged. Do not declare
`paginationOpts` in `args`:

`confect/notes.impl.ts` (excerpt)

```ts
const listMineByTag = FunctionImpl.make(
  databaseSchema,
  notes,
  "listMineByTag",
  ({ tag, paginationOpts }) =>
    Effect.gen(function* () {
      const { user } = yield* CurrentUser;
      const reader = yield* DatabaseReader;

      // Pass paginationOpts unchanged.
      return yield* reader
        .table("notes")
        .index(
          "by_authorId_and_tag",
          (q) => q.eq("authorId", user._id).eq("tag", tag),
          "desc",
        )
        .paginate(paginationOpts);
    }).pipe(Effect.orDie),
);
```

Full text search. Results come in relevance order:

`confect/messages.impl.ts` (imports for the excerpts below)

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Duration from "effect/Duration";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import refs from "./_generated/refs";
import databaseSchema from "./_generated/schema";
import {
  DatabaseReader,
  DatabaseWriter,
  MutationCtx,
  MutationRunner,
  QueryRunner,
  Scheduler,
} from "./_generated/services";
import messages, { RateLimited } from "./messages.spec";
import { rateLimiter } from "./rateLimits";
```

`confect/messages.impl.ts` (excerpt)

```ts
const search = FunctionImpl.make(
  databaseSchema,
  messages,
  "search",
  ({ channel, query }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;

      return yield* reader
        .table("messages")
        .search("search_body", (q) =>
          q.search("body", query).eq("channel", channel),
        )
        .take(10);
    }).pipe(Effect.orDie),
);
```

### E4. Writes

`insert` returns the new id. `patch` merges fields; an optional field set to
`undefined` is removed. `replace` replaces the document. `delete` removes it.

`confect/notes.impl.ts` (excerpt)

```ts
const create = FunctionImpl.make(
  databaseSchema,
  notes,
  "create",
  ({ text, tag }) =>
    Effect.gen(function* () {
      const { user } = yield* CurrentUser;
      const writer = yield* DatabaseWriter;

      return yield* writer.table("notes").insert({
        authorId: user._id,
        text,
        ...(tag === undefined ? {} : { tag }),
      });
    }).pipe(Effect.orDie),
);
```

`confect/notes.impl.ts` (excerpt)

```ts
const updateText = FunctionImpl.make(
  databaseSchema,
  notes,
  "updateText",
  ({ noteId, text }) =>
    Effect.gen(function* () {
      const writer = yield* DatabaseWriter;
      yield* getOwnedNote(noteId);

      yield* writer
        .table("notes")
        .patch(noteId, { text })
        .pipe(Effect.orDie);

      return null;
    }),
);
```

### E5. Auth and middleware

Read the identity with `Auth`. Use `identity.tokenIdentifier` as the stable
key. Never take a user id as an argument for authorization.

`confect/users.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "./_generated/id";

export class NotSignedIn extends Schema.TaggedError<NotSignedIn>()(
  "NotSignedIn",
  {},
) {}

export default GroupSpec.make().addFunction(
  FunctionSpec.publicMutation({
    name: "store",
    returns: () => Id("users"),
    error: () => NotSignedIn,
  }),
);
```

`confect/users.impl.ts`

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import * as Option from "effect/Option";

import databaseSchema from "./_generated/schema";
import { Auth, DatabaseReader, DatabaseWriter } from "./_generated/services";
import users, { NotSignedIn } from "./users.spec";

// Create the user row for the signed-in identity, or return the existing one.
const store = FunctionImpl.make(databaseSchema, users, "store", () =>
  Effect.gen(function* () {
    const auth = yield* Auth;
    const reader = yield* DatabaseReader;
    const writer = yield* DatabaseWriter;

    const identity = yield* auth.getUserIdentity.pipe(
      Effect.catchTag("NoUserIdentityFoundError", () =>
        Effect.fail(new NotSignedIn()),
      ),
    );

    const existing = yield* reader
      .table("users")
      .index("by_tokenIdentifier", (q) =>
        q.eq("tokenIdentifier", identity.tokenIdentifier),
      )
      .first()
      .pipe(Effect.orDie);

    if (Option.isSome(existing)) {
      return existing.value._id;
    }

    return yield* writer
      .table("users")
      .insert({
        tokenIdentifier: identity.tokenIdentifier,
        name: identity.name ?? "Anonymous",
        ...(identity.email === undefined ? {} : { email: identity.email }),
      })
      .pipe(Effect.orDie);
  }),
);

export default GroupImpl.make(databaseSchema, users).pipe(
  Layer.provide(store),
  GroupImpl.finalize,
);
```

Middleware replaces custom function wrappers. It runs after the args are
decoded and before the handler. It can provide a service, fail with a typed
error, or wrap the handler. Its error joins the error union of every function
it covers.

`confect/middleware/RequireUser.spec.ts`

```ts
import { MiddlewareSpec } from "@confect/core";
import * as Context from "effect/Context";
import * as Schema from "effect/Schema";

import type users from "../_generated/tables/users";

export class CurrentUser extends Context.Service<
  CurrentUser,
  { readonly user: typeof users.Doc.Type }
>()("confect/middleware/RequireUser.spec/CurrentUser") {}

export class Unauthenticated extends Schema.TaggedError<Unauthenticated>()(
  "Unauthenticated",
  {},
) {}

export default class RequireUser extends MiddlewareSpec.MiddlewareSpec<
  RequireUser,
  { provides: CurrentUser }
>()("RequireUser", {
  error: () => Unauthenticated,
  functionTypes: { query: true, mutation: true, action: false },
}) {}
```

`confect/middleware/RequireUser.impl.ts`

```ts
import { MiddlewareImpl } from "@confect/server";
import * as Effect from "effect/Effect";

import databaseSchema from "../_generated/schema";
import { Auth, DatabaseReader } from "../_generated/services";
import RequireUser, { CurrentUser, Unauthenticated } from "./RequireUser.spec";

export default MiddlewareImpl.provides(
  databaseSchema,
  RequireUser,
  CurrentUser,
  Effect.gen(function* () {
    const auth = yield* Auth;
    const reader = yield* DatabaseReader;

    const identity = yield* auth.getUserIdentity.pipe(
      Effect.catchTag("NoUserIdentityFoundError", () =>
        Effect.fail(new Unauthenticated()),
      ),
    );

    // Look the user up by tokenIdentifier, never by a caller-supplied id.
    const user = yield* reader
      .table("users")
      .get("by_tokenIdentifier", identity.tokenIdentifier)
      .pipe(
        Effect.catchTags({
          GetByIndexFailure: () => Effect.fail(new Unauthenticated()),
          DocumentDecodeError: Effect.die,
        }),
      );

    return { user };
  }),
);
```

Middleware rules:

- `functionTypes` must list `query`, `mutation`, and `action` with literal
  `true` or `false`.
- A group attaches middleware with `GroupSpec.make().middleware(Spec)`. One
  function attaches it with `FunctionSpec.publicQuery({...}).middleware(Spec)`.
- Middleware runs in attachment order. Group middleware runs before function
  middleware.
- Middleware does not apply to subgroups.
- Middleware cannot cover plain Convex functions. Do the auth check inside
  those functions.
- Provide the middleware impl in the group impl layer. `GroupImpl.finalize`
  fails to typecheck without it.
- A middleware that covers actions cannot read the database directly. Use
  `MiddlewareImpl.makeByFunctionType` and `runQuery` in the action branch.

### E6. Calling other functions

Refs come from the generated `refs` module. They carry the args, returns,
and error schemas, so you do not need the same-file return type annotation
that plain Convex needs.

`confect/messages.spec.ts`

```ts
import { FunctionSpec, GenericId, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "./_generated/id";
import messages from "./_generated/tables/messages";

export class RateLimited extends Schema.TaggedError<RateLimited>()(
  "RateLimited",
  { retryAfterMs: Schema.Finite },
) {}

export default GroupSpec.make()
  .addFunction(
    FunctionSpec.publicMutation({
      name: "send",
      args: () => ({
        channel: Schema.String,
        author: Schema.String,
        body: Schema.String,
      }),
      returns: () => Id("messages"),
      error: () => RateLimited,
    }),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "listByChannel",
      args: () => ({ channel: Schema.String }),
      returns: () => Schema.Array(messages.Doc),
    }),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "search",
      args: () => ({ channel: Schema.String, query: Schema.String }),
      returns: () => Schema.Array(messages.Doc),
    }),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "remindLater",
      args: () => ({ channel: Schema.String, body: Schema.String }),
      returns: () => GenericId.GenericId("_scheduled_functions"),
    }),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "cancelReminder",
      args: () => ({ scheduledId: GenericId.GenericId("_scheduled_functions") }),
      returns: () => Schema.Null,
    }),
  )
  .addFunction(
    FunctionSpec.internalQuery({
      name: "latestBodies",
      args: () => ({ channel: Schema.String }),
      returns: () => Schema.Array(Schema.String),
    }),
  )
  .addFunction(
    FunctionSpec.internalMutation({
      name: "postSystem",
      args: () => ({ channel: Schema.String, body: Schema.String }),
      returns: () => Id("messages"),
    }),
  )
  .addFunction(
    FunctionSpec.internalMutation({
      name: "postBounded",
      args: () => ({ channel: Schema.String, body: Schema.String }),
      returns: () => Schema.NullOr(Id("messages")),
    }),
  )
  .addFunction(
    FunctionSpec.publicAction({
      name: "echoLatest",
      args: () => ({ channel: Schema.String }),
      returns: () => Schema.NullOr(Id("messages")),
    }),
  );
```

In an action, use the runners. Each `runQuery` and `runMutation` call is a
separate transaction. Keep these calls few. Call `runAction` only to change
runtime (for example, to reach a Node action).

`confect/messages.impl.ts` (excerpt)

```ts
const echoLatest = FunctionImpl.make(
  databaseSchema,
  messages,
  "echoLatest",
  ({ channel }) =>
    Effect.gen(function* () {
      const { runQuery } = yield* QueryRunner;
      const { runMutation } = yield* MutationRunner;

      const [latest] = yield* runQuery(refs.internal.messages.latestBodies, {
        channel,
      });
      if (latest === undefined) {
        return null;
      }

      return yield* runMutation(refs.internal.messages.postSystem, {
        channel,
        body: `echo: ${latest}`,
      });
    }).pipe(Effect.orDie),
);
```

A nested `runMutation` in a mutation is a subtransaction. `transactionLimits`
can only make the limits smaller. When the nested call fails, its writes roll
back and the caller keeps its own writes. A declared error of the nested
function comes back decoded; catch it with `Effect.catchTag`. Any other
failure comes back as a defect; catch it with `Effect.catchDefect`.

`confect/messages.impl.ts` (excerpt)

```ts
const postBounded = FunctionImpl.make(
  databaseSchema,
  messages,
  "postBounded",
  ({ channel, body }) =>
    Effect.gen(function* () {
      const { runMutation } = yield* MutationRunner;

      return yield* runMutation(
        refs.internal.messages.postSystem,
        { channel, body },
        { transactionLimits: { documentsWritten: 1 } },
      ).pipe(Effect.catchDefect(() => Effect.succeed(null)));
    }).pipe(Effect.orDie),
);
```

Components. Mount the component in `convex/convex.config.ts`. Get the
component reference from `confect/_generated/components`, not from
`convex/_generated/api`. Component clients take the raw Convex ctx:

`convex/convex.config.ts`

```ts
import rateLimiter from "@convex-dev/rate-limiter/convex.config";
import { defineApp } from "convex/server";

const app = defineApp();
app.use(rateLimiter);

export default app;
```

`confect/rateLimits.ts`

```ts
import { MINUTE, RateLimiter } from "@convex-dev/rate-limiter";

import { components } from "./_generated/components";

// Use the Confect components registry, not convex/_generated/api.
export const rateLimiter = new RateLimiter(components.rateLimiter, {
  sendMessage: { kind: "token bucket", rate: 10, period: MINUTE, capacity: 3 },
});
```

`confect/messages.impl.ts` (excerpt)

```ts
const send = FunctionImpl.make(
  databaseSchema,
  messages,
  "send",
  ({ channel, author, body }) =>
    Effect.gen(function* () {
      // Component clients take the raw Convex ctx. The component write is part
      // of this mutation's transaction.
      const ctx = yield* MutationCtx;
      const status = yield* Effect.promise(() =>
        rateLimiter.limit(ctx, "sendMessage", { key: author }),
      );
      if (!status.ok) {
        return yield* new RateLimited({ retryAfterMs: status.retryAfter });
      }

      const writer = yield* DatabaseWriter;
      return yield* writer
        .table("messages")
        .insert({ channel, author, body })
        .pipe(Effect.orDie);
    }),
);
```

The component write is part of the calling mutation's transaction. Do the
auth check in your function before you call the component.

### E7. Scheduling and batches

`runAfter` takes an Effect `Duration`. `runAt` takes an Effect `DateTime`.
Both return an id in the `_scheduled_functions` table.

`confect/messages.impl.ts` (excerpt)

```ts
const remindLater = FunctionImpl.make(
  databaseSchema,
  messages,
  "remindLater",
  ({ channel, body }) =>
    Effect.gen(function* () {
      const scheduler = yield* Scheduler;

      return yield* scheduler.runAfter(
        Duration.minutes(5),
        refs.internal.messages.postSystem,
        { channel, body },
      );
    }),
);
```

`confect/messages.impl.ts` (excerpt)

```ts
const cancelReminder = FunctionImpl.make(
  databaseSchema,
  messages,
  "cancelReminder",
  ({ scheduledId }) =>
    Effect.gen(function* () {
      const scheduler = yield* Scheduler;
      yield* scheduler.cancel(scheduledId);
      return null;
    }),
);
```

Queries do not read the clock. Pass the time in as an argument:

`confect/tasks.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "./_generated/id";
import tasks from "./_generated/tables/tasks";

export default GroupSpec.make()
  .addFunction(
    FunctionSpec.publicMutation({
      name: "create",
      args: () => ({ text: Schema.String, dueAt: Schema.Finite }),
      returns: () => Id("tasks"),
    }),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "listOverdue",
      // The client passes the current time. Queries do not read the clock.
      args: () => ({ now: Schema.Finite }),
      returns: () => Schema.Array(tasks.Doc),
    }),
  )
  .addFunction(
    FunctionSpec.internalMutation({
      name: "purgeCompleted",
      returns: () => Schema.Null,
    }),
  );
```

`confect/tasks.impl.ts` (imports for the excerpts below)

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Duration from "effect/Duration";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import refs from "./_generated/refs";
import databaseSchema from "./_generated/schema";
import {
  DatabaseReader,
  DatabaseWriter,
  Scheduler,
  TransactionMetadata,
} from "./_generated/services";
import tasks from "./tasks.spec";
```

`confect/tasks.impl.ts` (excerpt)

```ts
const listOverdue = FunctionImpl.make(
  databaseSchema,
  tasks,
  "listOverdue",
  ({ now }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;

      // Put every predicate the index supports into the index range.
      return yield* reader
        .table("tasks")
        .index("by_isCompleted_and_dueAt", (q) =>
          q.eq("isCompleted", false).lt("dueAt", now),
        )
        .take(100);
    }).pipe(Effect.orDie),
);
```

A mutation that has more work than one transaction can hold does one batch,
then schedules itself with `Duration.zero`. Check the remaining limits with
`TransactionMetadata.getMetrics()`:

`confect/tasks.impl.ts` (excerpt)

```ts
const BATCH_SIZE = 500;
const WRITE_RESERVE = 50;

// Delete in batches. Schedule a continuation before the transaction limits.
const purgeCompleted = FunctionImpl.make(
  databaseSchema,
  tasks,
  "purgeCompleted",
  () =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;
      const writer = yield* DatabaseWriter;
      const transaction = yield* TransactionMetadata;
      const scheduler = yield* Scheduler;

      const continueLater = scheduler.runAfter(
        Duration.zero,
        refs.internal.tasks.purgeCompleted,
      );

      const batch = yield* reader
        .table("tasks")
        .index("by_isCompleted", (q) => q.eq("isCompleted", true))
        .take(BATCH_SIZE);

      for (const task of batch) {
        yield* writer.table("tasks").delete(task._id);

        const metrics = yield* transaction.getMetrics();
        if (metrics.documentsWritten.remaining <= WRITE_RESERVE) {
          yield* continueLater;
          return null;
        }
      }

      if (batch.length === BATCH_SIZE) {
        yield* continueLater;
      }
      return null;
    }).pipe(Effect.orDie),
);
```

### E8. File storage

`getUrl` and `generateUploadUrl` produce a `URL`. Return it with
`Schema.URLFromString`. Read file metadata from the `_storage` system table.
Do not use the deprecated `getMetadata`.

`confect/files.spec.ts`

```ts
import { FunctionSpec, GenericId, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "./_generated/id";

export default GroupSpec.make()
  .addFunction(
    FunctionSpec.publicMutation({
      name: "generateUploadUrl",
      returns: () => Schema.URLFromString,
    }),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "save",
      args: () => ({ name: Schema.String, storageId: GenericId.GenericId("_storage") }),
      returns: () => Id("files"),
    }),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "getUrl",
      args: () => ({ fileId: Id("files") }),
      returns: () => Schema.NullOr(Schema.URLFromString),
    }),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "metadata",
      args: () => ({ storageId: GenericId.GenericId("_storage") }),
      returns: () =>
        Schema.NullOr(
          Schema.Struct({
            size: Schema.Finite,
            contentType: Schema.optionalKey(Schema.String),
          }),
        ),
    }),
  )
  .addFunction(
    FunctionSpec.internalAction({
      name: "storeText",
      args: () => ({ text: Schema.String }),
      returns: () => GenericId.GenericId("_storage"),
    }),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "remove",
      args: () => ({ fileId: Id("files") }),
      returns: () => Schema.Null,
    }),
  );
```

`confect/files.impl.ts`

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import databaseSchema from "./_generated/schema";
import { DatabaseReader, DatabaseWriter, Storage } from "./_generated/services";
import files from "./files.spec";

const generateUploadUrl = FunctionImpl.make(
  databaseSchema,
  files,
  "generateUploadUrl",
  () =>
    Effect.gen(function* () {
      const storage = yield* Storage;
      // An Effect property: no parentheses.
      return yield* storage.generateUploadUrl;
    }),
);

const save = FunctionImpl.make(
  databaseSchema,
  files,
  "save",
  ({ name, storageId }) =>
    Effect.gen(function* () {
      const writer = yield* DatabaseWriter;
      return yield* writer.table("files").insert({ name, storageId });
    }).pipe(Effect.orDie),
);

const getUrl = FunctionImpl.make(databaseSchema, files, "getUrl", ({ fileId }) =>
  Effect.gen(function* () {
    const reader = yield* DatabaseReader;
    const storage = yield* Storage;

    const file = yield* reader.table("files").get(fileId);
    return yield* storage
      .getUrl(file.storageId)
      .pipe(Effect.catchTag("BlobNotFoundError", () => Effect.succeed(null)));
  }).pipe(
    Effect.catchTag("GetByIdFailure", () => Effect.succeed(null)),
    Effect.orDie,
  ),
);

// File metadata lives in the _storage system table.
// Do not use the deprecated storage getMetadata call.
const metadata = FunctionImpl.make(
  databaseSchema,
  files,
  "metadata",
  ({ storageId }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;
      const stored = yield* reader.table("_storage").get(storageId);

      return {
        size: stored.size,
        ...(stored.contentType === undefined
          ? {}
          : { contentType: stored.contentType }),
      };
    }).pipe(
      Effect.catchTag("GetByIdFailure", () => Effect.succeed(null)),
      Effect.orDie,
    ),
);

// get and store need an action (or an HTTP action).
const storeText = FunctionImpl.make(
  databaseSchema,
  files,
  "storeText",
  ({ text }) =>
    Effect.gen(function* () {
      const storage = yield* Storage;
      return yield* storage.store(new Blob([text], { type: "text/plain" }));
    }),
);

const remove = FunctionImpl.make(databaseSchema, files, "remove", ({ fileId }) =>
  Effect.gen(function* () {
    const reader = yield* DatabaseReader;
    const writer = yield* DatabaseWriter;
    const storage = yield* Storage;

    const file = yield* reader.table("files").get(fileId);
    yield* storage.delete(file.storageId);
    yield* writer.table("files").delete(fileId);
    return null;
  }).pipe(Effect.orDie),
);

export default GroupImpl.make(databaseSchema, files).pipe(
  Layer.provide(
    Layer.mergeAll(generateUploadUrl, save, getUrl, metadata, storeText, remove),
  ),
  GroupImpl.finalize,
);
```

### E9. Vector search

`VectorSearch` exists only in actions. It returns `{ _id, _score }` pairs, in
descending score order. Hydrate them through an internal query and keep the
order. The filter supports only equality on the declared `filterFields`, and
`q.or(...)`.

`confect/documents.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "./_generated/id";
import documents from "./_generated/tables/documents";

export default GroupSpec.make()
  .addFunction(
    FunctionSpec.internalQuery({
      name: "getMany",
      args: () => ({ ids: Schema.Array(Id("documents")) }),
      returns: () => Schema.Array(documents.Doc),
    }),
  )
  .addFunction(
    FunctionSpec.publicAction({
      name: "similar",
      args: () => ({
        embedding: Schema.Array(Schema.Finite),
        category: Schema.String,
      }),
      returns: () =>
        Schema.Array(
          Schema.Struct({ document: documents.Doc, score: Schema.Finite }),
        ),
    }),
  );
```

`confect/documents.impl.ts`

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import * as Option from "effect/Option";

import refs from "./_generated/refs";
import databaseSchema from "./_generated/schema";
import { DatabaseReader, QueryRunner, VectorSearch } from "./_generated/services";
import documents from "./documents.spec";

const getMany = FunctionImpl.make(
  databaseSchema,
  documents,
  "getMany",
  ({ ids }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;
      const found = yield* Effect.forEach(ids, (id) =>
        reader.table("documents").get(id).pipe(Effect.option),
      );
      return found.flatMap(Option.toArray);
    }),
);

// VectorSearch exists only in actions. It returns { _id, _score } pairs.
// Hydrate through an internal query and keep the search order.
const similar = FunctionImpl.make(
  databaseSchema,
  documents,
  "similar",
  ({ embedding, category }) =>
    Effect.gen(function* () {
      const vectorSearch = yield* VectorSearch;
      const { runQuery } = yield* QueryRunner;

      const hits = yield* vectorSearch("documents", "by_embedding", {
        vector: [...embedding],
        limit: 10,
        filter: (q) => q.eq("category", category),
      });

      const docs = yield* runQuery(refs.internal.documents.getMany, {
        ids: hits.map((hit) => hit._id),
      });
      const byId = new Map(docs.map((doc) => [doc._id, doc]));

      return hits.flatMap((hit) => {
        const document = byId.get(hit._id);
        return document === undefined ? [] : [{ document, score: hit._score }];
      });
    }).pipe(Effect.orDie),
);

export default GroupImpl.make(databaseSchema, documents).pipe(
  Layer.provide(Layer.mergeAll(getMany, similar)),
  GroupImpl.finalize,
);
```

### E10. Environment variables

Confect installs a `ConfigProvider` for the Convex runtime, so `Config` reads
Convex environment variables. `CONVEX_SITE_URL` and `CONVEX_CLOUD_URL` are
always set. Node actions use Effect's default provider.

`confect/settings.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

export default GroupSpec.make().addFunction(
  FunctionSpec.internalAction({
    name: "describe",
    returns: () =>
      Schema.Struct({ siteUrl: Schema.String, maxResults: Schema.Finite }),
  }),
);
```

`confect/settings.impl.ts`

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import databaseSchema from "./_generated/schema";
import settings from "./settings.spec";

// Confect installs a Convex-aware ConfigProvider. Read env vars with Config.
const describe = FunctionImpl.make(databaseSchema, settings, "describe", () =>
  Effect.gen(function* () {
    const siteUrl = yield* Config.String("CONVEX_SITE_URL");
    const maxResults = yield* Config.Int("MAX_RESULTS").pipe(
      Config.withDefault(20),
    );
    return { siteUrl, maxResults };
  }).pipe(Effect.orDie),
);

export default GroupImpl.make(databaseSchema, settings).pipe(
  Layer.provide(describe),
  GroupImpl.finalize,
);
```

`Config.Array` and `Config.Record` read one delimited variable (for example
`A=1,B=2`). The Convex provider cannot assemble a collection from several
variables.

### E11. Node actions

A Node group holds only Node actions. Keep queries and mutations in other
groups. `fetch` works in the default runtime, so you do not need Node for it.

`confect/digest.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

// makeNode() marks the group as Node runtime. Codegen adds "use node".
// A Node group holds only Node actions.
export default GroupSpec.makeNode().addFunction(
  FunctionSpec.internalNodeAction({
    name: "sha256",
    args: () => ({ text: Schema.String }),
    returns: () => Schema.String,
  }),
);
```

`confect/digest.impl.ts`

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import { createHash } from "node:crypto";

import databaseSchema from "./_generated/schema";
import digest from "./digest.spec";

const sha256 = FunctionImpl.make(databaseSchema, digest, "sha256", ({ text }) =>
  Effect.sync(() => createHash("sha256").update(text).digest("hex")),
);

export default GroupImpl.make(databaseSchema, digest).pipe(
  Layer.provide(sha256),
  GroupImpl.finalize,
);
```

### E12. Plain Convex functions and escape hatches

Some libraries and components need ordinary Convex functions. Write them in
`<group>.ts` with `convex/server` generic builders and Convex validators.
Name them in the spec with a type-only import. The impl passes the function
value through.

`confect/maintenance.ts`

```ts
import { internalMutationGeneric } from "convex/server";
import { v } from "convex/values";

// A plain Convex function, for libraries or components that need one.
// Confect does not decode or encode its args. Use Convex validators.
export const clearResults = internalMutationGeneric({
  args: { limit: v.number() },
  returns: v.number(),
  handler: async (ctx, { limit }) => {
    const rows = await ctx.db.query("results").take(limit);
    for (const row of rows) {
      await ctx.db.delete("results", row._id);
    }
    return rows.length;
  },
});
```

`confect/maintenance.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "./_generated/id";
import type { clearResults } from "./maintenance";

export default GroupSpec.make()
  // Type-only import: the spec ships to the client.
  .addFunction(
    FunctionSpec.convexInternalMutation<typeof clearResults>()("clearResults"),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "normalizeNoteId",
      args: () => ({ raw: Schema.String }),
      returns: () => Schema.NullOr(Id("notes")),
    }),
  )
  .addFunction(
    FunctionSpec.internalMutation({
      name: "clearResultsHandle",
      returns: () => Schema.String,
    }),
  );
```

`confect/maintenance.impl.ts`

```ts
import { Ref } from "@confect/core";
import { FunctionImpl, GroupImpl } from "@confect/server";
import { createFunctionHandle } from "convex/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import refs from "./_generated/refs";
import databaseSchema from "./_generated/schema";
import { QueryCtx } from "./_generated/services";
import { clearResults } from "./maintenance";
import maintenance from "./maintenance.spec";

// A plain Convex function's impl is the Convex function value itself.
const clearResultsImpl = FunctionImpl.make(
  databaseSchema,
  maintenance,
  "clearResults",
  clearResults,
);

// Escape hatch: Confect does not wrap db.normalizeId. Use the raw ctx.
const normalizeNoteId = FunctionImpl.make(
  databaseSchema,
  maintenance,
  "normalizeNoteId",
  ({ raw }) =>
    Effect.gen(function* () {
      const ctx = yield* QueryCtx;
      return ctx.db.normalizeId("notes", raw);
    }),
);

// Escape hatch: mint a function handle to pass across a component boundary.
const clearResultsHandle = FunctionImpl.make(
  databaseSchema,
  maintenance,
  "clearResultsHandle",
  () =>
    Effect.promise(() =>
      createFunctionHandle(
        Ref.getFunctionReference(refs.internal.maintenance.clearResults),
      ),
    ),
);

export default GroupImpl.make(databaseSchema, maintenance).pipe(
  Layer.provide(
    Layer.mergeAll(clearResultsImpl, normalizeNoteId, clearResultsHandle),
  ),
  GroupImpl.finalize,
);
```

### E13. HTTP actions

`HttpRouter.make` registers one catch-all HTTP action under `/`. The Effect
router owns the paths. Endpoint paths are exact; add a prefix with
`.prefix(...)`. The payload schema decodes the body, and a body that fails
decoding gets a 400 response. Commit effects through an internal mutation.

`confect/http/MessagesApi.ts`

```ts
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";
import * as Schema from "effect/Schema";
import * as HttpApi from "effect/http-api/HttpApi";
import * as HttpApiBuilder from "effect/http-api/HttpApiBuilder";
import * as HttpApiEndpoint from "effect/http-api/HttpApiEndpoint";
import * as HttpApiGroup from "effect/http-api/HttpApiGroup";

import { Id } from "../_generated/id";
import refs from "../_generated/refs";
import { MutationRunner, QueryRunner } from "../_generated/services";
import messages from "../_generated/tables/messages";

class MessagesGroup extends HttpApiGroup.make("messages")
  .add(
    HttpApiEndpoint.get("list", "/messages", {
      query: { channel: Schema.String },
      success: Schema.Array(messages.Doc),
    }),
  )
  .add(
    // The payload schema decodes the body. A bad body gets a 400 response.
    HttpApiEndpoint.post("post", "/messages", {
      payload: Schema.Struct({ channel: Schema.String, body: Schema.String }),
      success: Id("messages"),
    }),
  ) {}

export class Api extends HttpApi.make("Api").add(MessagesGroup).prefix("/api") {}

const MessagesLive = HttpApiBuilder.group(Api, "messages", (handlers) =>
  handlers
    .handle("list", ({ query }) =>
      Effect.gen(function* () {
        const { runQuery } = yield* QueryRunner;
        return yield* runQuery(refs.public.messages.listByChannel, {
          channel: query.channel,
        });
      }).pipe(Effect.orDie),
    )
    .handle("post", ({ payload }) =>
      Effect.gen(function* () {
        // Commit effects through an internal mutation, not a public one.
        const { runMutation } = yield* MutationRunner;
        return yield* runMutation(refs.internal.messages.postSystem, payload);
      }).pipe(Effect.orDie),
    ),
);

export const layer = HttpApiBuilder.layer(Api).pipe(Layer.provide(MessagesLive));
```

`confect/http.ts`

```ts
import { HttpRouter as ConfectHttpRouter } from "@confect/server";
import { httpActionGeneric } from "convex/server";
import * as Layer from "effect/Layer";
import * as HttpRouter from "effect/http/HttpRouter";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

import * as MessagesApi from "./http/MessagesApi";

const http = ConfectHttpRouter.make(
  Layer.mergeAll(
    MessagesApi.layer,
    HttpRouter.add("GET", "/health", HttpServerResponse.text("OK")),
  ),
);

// Escape hatch: a plain Convex route. Exact paths match before the catch-all.
http.route({
  path: "/echo",
  method: "POST",
  handler: httpActionGeneric(async (_ctx, request) => {
    const body = await request.bytes();
    return new Response(body, { status: 200 });
  }),
});

export default http;
```

### E14. Auth config

The `domain` is the issuer URL of the JWT provider. The `applicationID` must
equal the JWT `aud` claim. Without this file, `auth.getUserIdentity` always
fails with `NoUserIdentityFoundError`.

`confect/auth.ts`

```ts
// Codegen copies this file to convex/auth.config.ts unchanged.
export default {
  providers: [
    {
      domain: "https://your-auth-provider.example.com",
      applicationID: "convex",
    },
  ],
};
```

On the client, use `ConvexProviderWithAuth`, not `ConvexProvider`, when you
use auth.

### E15. Cron jobs

A cron targets an internal mutation or action. A `Cron` schedule has minute
granularity and no seconds field. A `Duration` interval must be a whole
number of seconds, minutes, or hours. Convex runs schedules in UTC.

`confect/crons.ts`

```ts
import { CronJob, CronJobs } from "@confect/server";
import * as Cron from "effect/Cron";
import * as Duration from "effect/Duration";

import refs from "./_generated/refs";

export default CronJobs.make()
  .add(
    CronJob.make(
      "purge completed tasks",
      Cron.parseUnsafe("0 3 * * *"),
      refs.internal.tasks.purgeCompleted,
    ),
  )
  .add(
    CronJob.make(
      "post hourly status",
      Duration.hours(1),
      refs.internal.messages.postSystem,
      { channel: "status", body: "Still running." },
    ),
  );
```

### E16. Testing

Build the layer once per test for a clean database. The module glob must
include the `convex/` modules and `convex/_generated/`. Codegen writes
`_generated` as `.js` plus `.d.ts`, so the glob matches `.ts` and `.js` and
excludes `.d.ts`.

`test/TestConfect.ts`

```ts
/// <reference types="vite/client" />

import { TestConfect as TestConfect_ } from "@confect/test";

import convexSchema from "../confect/_generated/convexSchema";
import confectSchema from "../confect/_generated/schema";

export const TestConfect = TestConfect_.TestConfect<typeof confectSchema>();

// A new test database each time the layer is built.
export const layer = TestConfect_.layer(
  confectSchema,
  convexSchema,
  import.meta.glob(["../convex/**/*.{ts,js}", "!../convex/**/*.d.ts"]),
);
```

`test/notes.test.ts`

```ts
import { describe, expect, it } from "@effect/vitest";
import * as Effect from "effect/Effect";

import { Id } from "../confect/_generated/id";
import refs from "../confect/_generated/refs";
import { DatabaseWriter } from "../confect/_generated/services";
import * as TestConfect from "./TestConfect";

const alice = {
  subject: "alice",
  issuer: "https://auth.example.com",
  tokenIdentifier: "https://auth.example.com|alice",
  name: "Alice",
};

describe("notes", () => {
  it.effect("creates and lists notes for the signed-in user", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;
      const asAlice = confect.withIdentity(alice);

      yield* asAlice.mutation(refs.public.users.store);
      yield* asAlice.mutation(refs.public.notes.create, { text: "Hello" });

      const notes = yield* asAlice.query(refs.public.notes.listMine);
      expect(notes.map((note) => note.text)).toEqual(["Hello"]);

      const page = yield* asAlice.query(refs.public.notes.listMineByTag, {
        tag: "work",
        paginationOpts: { numItems: 10, cursor: null },
      });
      expect(page.page).toHaveLength(0);
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

  it.effect("hides notes that belong to another user", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;

      // Seed data directly with mutation-context services.
      const noteId = yield* confect.run(
        Effect.gen(function* () {
          const writer = yield* DatabaseWriter;
          const ownerId = yield* writer.table("users").insert({
            tokenIdentifier: "https://auth.example.com|bob",
            name: "Bob",
          });
          return yield* writer
            .table("notes")
            .insert({ authorId: ownerId, text: "Private" });
        }).pipe(Effect.orDie),
        Id("notes"),
      );

      const asAlice = confect.withIdentity(alice);
      yield* asAlice.mutation(refs.public.users.store);

      const result = yield* asAlice
        .query(refs.public.notes.get, { noteId })
        .pipe(
          Effect.catchTag("NoteNotFound", (error) =>
            Effect.succeed(`missing ${error.noteId}`),
          ),
          Effect.orDie,
        );

      expect(result).toBe(`missing ${noteId}`);
    }).pipe(Effect.provide(TestConfect.layer)),
  );
});
```

`test/messages.test.ts`

```ts
import { describe, expect, it } from "@effect/vitest";
import * as Effect from "effect/Effect";

import refs from "../confect/_generated/refs";
import { DatabaseWriter } from "../confect/_generated/services";
import * as TestConfect from "./TestConfect";

describe("messages", () => {
  it.effect("an action reads and writes through the runners", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;

      yield* confect.mutation(refs.internal.messages.postSystem, {
        channel: "general",
        body: "hi",
      });
      const echoed = yield* confect.action(refs.public.messages.echoLatest, {
        channel: "general",
      });
      expect(echoed).not.toBeNull();

      const bodies = yield* confect.query(refs.internal.messages.latestBodies, {
        channel: "general",
      });
      expect(bodies).toEqual(["echo: hi", "hi"]);
    }).pipe(Effect.provide(TestConfect.layer)),
  );

  it.effect("a bounded nested mutation commits within its limit", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;

      const messageId = yield* confect.mutation(
        refs.internal.messages.postBounded,
        { channel: "general", body: "bounded" },
      );
      expect(messageId).not.toBeNull();
    }).pipe(Effect.provide(TestConfect.layer)),
  );

  it.effect("the HTTP API decodes the body and rejects a bad one", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;

      const created = yield* confect.fetch("/api/messages", {
        method: "POST",
        headers: { "content-type": "application/json" },
        body: JSON.stringify({ channel: "general", body: "from http" }),
      });
      expect(created.status).toBe(200);

      const rejected = yield* confect.fetch("/api/messages", {
        method: "POST",
        headers: { "content-type": "application/json" },
        body: JSON.stringify({ channel: 42 }),
      });
      expect(rejected.status).toBe(400);

      const listed = yield* confect.fetch("/api/messages?channel=general");
      const body = yield* Effect.promise(() => listed.json());
      expect(body).toHaveLength(1);
    }).pipe(Effect.provide(TestConfect.layer)),
  );

  it.effect("purgeCompleted deletes completed tasks only", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;

      yield* confect.run(
        Effect.gen(function* () {
          const writer = yield* DatabaseWriter;
          yield* writer
            .table("tasks")
            .insert({ text: "done", isCompleted: true, dueAt: 0 });
          yield* writer
            .table("tasks")
            .insert({ text: "open", isCompleted: false, dueAt: 0 });
        }).pipe(Effect.orDie),
      );

      yield* confect.mutation(refs.internal.tasks.purgeCompleted);

      const overdue = yield* confect.query(refs.public.tasks.listOverdue, {
        now: 1,
      });
      expect(overdue.map((task) => task.text)).toEqual(["open"]);
    }).pipe(Effect.provide(TestConfect.layer)),
  );
});
```

Test notes:

- Install `@confect/test`, `convex-test`, and `@effect/vitest` as dev
  dependencies.
- Inline `convex-test` in the Vitest config (`server.deps.inline`).
- Add `/// <reference types="vite/client" />` only to files that call
  `import.meta.glob`.
- Do not add a `compilerOptions.types` entry for a package that is not
  installed. An unresolved entry fails with TS2688.
- A component that the code under test calls must also be registered with
  `convex-test`. The examples above do not call the rate limiter for this
  reason.

## Platform rules that still apply

Confect does not change these Convex rules. Apply them to Confect code.

Functions:

- Use internal functions for anything that only your own code calls. A
  public function is on the public Internet.
- Validate every argument. In Confect, an omitted `args` means "no
  arguments", and Confect still validates that.
- Queries and mutations are transactions. Convex uses optimistic
  concurrency control and retries a mutation that conflicts. Many writes to
  the same document cause conflicts.
- Actions have no database access. They use the runners.
- `undefined` is not a Convex value. Return `null`.

Values and documents:

- Arrays hold at most 8192 values. Objects hold at most 1024 entries.
- Strings, bytes, and documents must be smaller than 1 MB.
- Field names must be non-empty and must not start with `$` or `_`.
- Record keys must be ASCII, non-empty, and must not start with `$` or `_`.
- Do not store unbounded lists in one document. Use a child table with a
  foreign key.
- Keep high-churn data (heartbeats, presence) in its own table.

Indexes and queries:

- Query index fields in the order of the index. Make another index for
  another order.
- Convex adds `_creationTime` as the last index field. Rely on it as the tie
  breaker.
- Order is ascending by default.
- Put every predicate that the index supports in the index range. A filter
  after the scan does not reduce the rows read.
- Return bounded results (`take`, `paginate`), unless the user asks for all
  rows.
- There is no count operator. Keep a counter document, or use
  `@convex-dev/aggregate` and update it in the same mutation as each write.
- There is no query delete. Read a batch and delete each document.
- When work does not fit in one transaction, do one batch and schedule the
  rest.
- A new index on a large table blocks the deploy until backfill completes.

Time:

- Do not read the wall clock in a query. Pass the time as an argument, or
  keep a status field that a scheduled mutation updates. Reading the clock is
  fine in mutations and actions.

Auth:

- Use `tokenIdentifier` for identity lookups and ownership checks. Do not use
  `subject` alone.
- Never accept a user id as an argument for authorization.

Components:

- App functions wrap component functions. Clients cannot call component
  functions.
- When a component mirrors a table, update it in the same mutation as every
  write to that table.
- A component mutation call is a subtransaction.
- Use `@convex-dev/rate-limiter` for quotas, `@convex-dev/agent` for LLM
  conversations, `@convex-dev/workpool` for bounded parallel work,
  `@convex-dev/presence` for presence, and `@convex-dev/aggregate` for counts
  and ranks. Do not hand-roll them.

Limits and cost:

- Read the current use and the remaining limits with
  `TransactionMetadata.getMetrics()`. The limit values are in the Convex docs
  (https://docs.convex.dev/production/state/limits).
- Bounded indexed reads, few action-to-function calls, and cache-friendly
  queries (no clock reads) keep cost down.

## Gaps

Confect 10.0.0 does not wrap these Convex features. Each entry gives the
supported escape hatch.

| Convex feature | Status in Confect 10.0.0 | Escape hatch |
| --- | --- | --- |
| Staged indexes (`staged: true`) | `Table.index(name, fields)` has no options argument. Codegen owns `convex/schema.ts`. | None. Plan for the deploy to block while the new index backfills. |
| `.filter()` on a query | The query builder has no `filter`. Only `paginate(opts, filter)` accepts a filter. | Put the predicate in the index range. For the rest, use `.stream()` with `Stream.filter` and `Stream.take`, or the `filter` argument of `paginate`. The rows read do not go down. |
| `for await` over a query | Not available. | `.stream()` gives an Effect `Stream`. |
| `ctx.db.normalizeId` | Not wrapped. | `const ctx = yield* QueryCtx` (or `MutationCtx`), then `ctx.db.normalizeId(table, raw)`. See [E12](#e12-plain-convex-functions-and-escape-hatches). |
| Component client calls | Component clients take a raw Convex ctx. | `yield* MutationCtx` (or `QueryCtx`, `ActionCtx`) and wrap the call in `Effect.promise`. See [E6](#e6-calling-other-functions). |
| `createFunctionHandle` and calling a `FunctionHandle` | Not wrapped. | `createFunctionHandle(Ref.getFunctionReference(ref))` from `convex/server` in `Effect.promise`. Call a received handle with the raw ctx (`ctx.runMutation(handle, args)`). See [E12](#e12-plain-convex-functions-and-escape-hatches). |
| Authoring a local component (`defineComponent`) | Codegen reads mounted components but does not build component code. | Write the component in plain Convex in its own directory. Call it from Confect through the raw ctx. |
| Middleware on plain Convex functions | Rejected at the type level and at run time. | Do the auth check inside the plain function. |
| Typed app env (`defineApp({ env })` with `env` from `convex/_generated/server`) | Not surfaced by Confect. | Read variables with `Config`. Decode them with `Config.schema` when you need a type. |
| Client code | Out of scope for this file. `@confect/react` and `@confect/js` were not in the check project. | Use `convex/react` with the Convex function names, or read the Confect client docs for the installed version. |

## Version notes

The installed 10.0.0 source differs from the current docs at confect.dev in
these places. Follow the source.

| Topic | Docs say | 10.0.0 source does |
| --- | --- | --- |
| `Id` helper | `Id("_storage")` and an `Id<"t">` type come from `confect/_generated/id`. | The generated `Id` accepts only your tables, and the module exports no `Id` type. Use `GenericId.GenericId("_storage")` and `GenericId.GenericId<"t">` from `@confect/core`. |
| `Date.now()` in a query | Returns `0`. | Not stubbed. A raw `Date.now()` reads the real time, and Convex evicts the query from the cache. Only Effect's internal clock reads (logs, spans) are stubbed. `Clock.currentTimeMillis` reads the real time. |
| `Effect.timed` in a query | Not stated. | Reports zero elapsed time, because the internal clock is stubbed. |

## Checklist

- [ ] Each table is one file in `confect/tables/` with a default `Table.make`.
- [ ] Index names list all fields in order.
- [ ] Numbers use `Schema.Finite`. Optional fields use `Schema.optionalKey`.
- [ ] Functions that return nothing use `returns: () => Schema.Null`.
- [ ] Specs value-import nothing from `@confect/server`.
- [ ] Internal work uses `FunctionSpec.internal*` and `refs.internal.*`.
- [ ] Expected failures have an `error` schema and use
      `return yield* new MyError(...)`. Other failures use `Effect.orDie`.
- [ ] Reads use an index and are bounded.
- [ ] Queries get the time as an argument.
- [ ] Auth uses `tokenIdentifier`, through `Auth` or middleware.
- [ ] Component calls use the raw ctx service and run in the same mutation as
      the table write.
- [ ] `confect codegen` passes, then `tsc --noEmit` passes.
