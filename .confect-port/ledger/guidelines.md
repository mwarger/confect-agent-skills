# Ledger: guidelines

- Source: Convex AI guidelines (`convex-guidelines.md`, 459 lines, Convex `^1.44.0`).
- Target: `skills/confect/references/guidelines.md` (new file).
- Frontmatter: none. The source has no frontmatter, so there is no `name` to change.
- Verification: every TypeScript block in the target (32 blocks) is copied verbatim from a file in a scratch check project. That project passed `confect codegen`, `tsc --noEmit` (strict, `exactOptionalPropertyTypes`), and Vitest (9 tests). Two blocks are unchanged upstream code that Confect does not touch (the client `tsx` block and `convex/convex.config.ts`).
- New verified facts (from package source, not in `confect-patterns.md`): `patch` fails with `GetByIdFailure` and reads the document first; `replace` and `delete` on a missing document are defects; `get(index, ...)` uses `.unique()`, so more than one match is a defect; `PaginationOptions` and `PaginationResult` field shapes; a redeclared `paginationOpts` fails to typecheck; `returns` is required; a Node group accepts only Node actions; `Storage` has no `getMetadata`; union tables get system fields on every member; `CronJobs` maps `Cron` to `crons.cron` and `Duration` to `crons.interval` only; codegen removes every `.ts` file under `convex/` that is not a group module (except `schema.ts`, `http.ts`, `crons.ts`, `auth.config.ts`, `convex.config.ts`, `_generated/`); `Schema.Record(Id(...), ...)` does not typecheck; `Struct.pick`, `Struct.omit`, `Struct.assign`, `Struct.map(Schema.optionalKey)` with `mapFields` compile.

Fix pass: codegen writes `convex/auth.config.ts` as a re-export (not a copy). Added 4 Bad/Good pairs (paginationOpts, components import, raw ctx, Record key). Illustrative snippets, not compiled.

Counts: kept 52, updated 111, added 31, removed 2.

| # | Source lines | Item | Action | Reason |
| --- | --- | --- | --- | --- |
| 1 | 1 | Title `# Convex guidelines` | Update | Renamed to `# Confect guidelines`. |
| 2 | 3 | Target version line | Update | Names Confect 10.0.0 packages and `effect` 4.0.0; keeps Convex `^1.44.0`. |
| 3 | (new) | Intro paragraph | Add | States that Confect changes code, not platform behavior, and that all platform rules apply. |
| 4 | (new) | Pointer to `confect-patterns.md` | Add | Full examples E1-E16 and gaps table live there. |
| 5 | (new) | Convex guidelines stay the platform source | Add | Task rule: keep Convex guidelines as the source for platform rules. |
| 6 | (new) | Package source is the API source | Add | Prevents guessed exports. |
| 7 | (new) | Compiled-code note | Add | Records how the examples were verified. |
| 8 | 5 | `## Function guidelines` | Keep | Heading still applies. |
| 9 | 7 | `### Http endpoint syntax` | Keep | Heading still applies. |
| 10 | 9 | `convex/http.ts` with `httpAction` | Update | Confect uses `confect/http.ts` and `HttpRouter.make`; codegen writes `convex/http.ts`. |
| 11 | 11-23 | `httpRouter` echo code | Update | Replaced with E13 `confect/http.ts`; the echo route stays as a plain `httpActionGeneric` escape hatch, so the sample still teaches the same route. |
| 12 | (new) | `payload` schema decodes the body; 400 on bad body | Add | Confect form of body validation, verified by the E16 HTTP test. |
| 13 | (new) | `MessagesGroup` excerpt | Add | Shows the payload schema in code. |
| 14 | 25 | Treat `req.json()` as `unknown` | Update | Scoped to plain `httpActionGeneric` routes; upstream wording kept for that case. |
| 15 | 26 | Exact path registration | Update | Adds the catch-all under `/`, exact endpoint paths, `.prefix`, and plain routes matching first. |
| 16 | 28 | `### Validators` | Keep | Heading still applies. |
| 17 | 30 | Array validator example intro | Update | Validators are Effect schemas compiled by codegen. |
| 18 | 32-44 | `v.array(v.union(...))` code | Update | Replaced with `Schema.Array(Schema.Union([...]))` in a `FunctionSpec` (compiled). |
| 19 | 46 | `v.object` `.pick/.omit/.partial/.extend` | Update | Effect v4 `mapFields` with `Struct.pick`, `Struct.omit`, `Struct.map(Schema.optionalKey)`, `Struct.assign`; table fields via `notes.Fields.fields`. |
| 20 | (new) | Contact derivation code | Add | Compiled example of the derivations. |
| 21 | (new) | `args: () => ContactName.fields` code | Add | Shows `.fields` supplying args. |
| 22 | 47 | `schema.doc` / `docValidator` | Update | `notes.Doc` from generated table module; union members get system fields (verified in `SystemFields.ts`). |
| 23 | 48 | Discriminated union intro | Update | Table file form instead of `defineSchema`. |
| 24 | 50-68 | `defineTable(v.union(...))` code | Update | Replaced with E1 `confect/tables/results.ts`. |
| 25 | 70 | Types table intro | Update | "validators" to "Effect schemas". |
| 26 | 71-72 | Table header | Update | Validator column renamed to Effect schema column. |
| 27 | 73 | Id row | Update | `Id(tableName)`; system tables use `GenericId.GenericId`. |
| 28 | 74 | Null row | Update | `Schema.Null`; notes kept verbatim; adds `returns: () => Schema.Null` and rejected `Undefined`/`Void`. |
| 29 | 75 | Int64 row | Update | `Schema.BigInt`; notes verbatim. |
| 30 | 76 | Float64 row | Update | `Schema.Finite`; notes verbatim; adds NaN/Infinity rule for `Finite` vs `Number`. |
| 31 | 77 | Boolean row | Update | `Schema.Boolean`. |
| 32 | 78 | String row | Update | `Schema.String`; notes verbatim; adds that checks run in Confect, not Convex. |
| 33 | 79 | Bytes row | Update | `Schema.instanceOf(ArrayBuffer)`; notes verbatim. |
| 34 | 80 | Array row | Update | `Schema.Array(values)`; notes verbatim. |
| 35 | 81 | Object row | Update | `Schema.Struct(...)`; notes verbatim. |
| 36 | 82-83 | Record row (outside the table after a blank line) | Update | Rejoined into the table; `Schema.Record(keys, values)`; notes verbatim; adds that `Id` is not a valid key schema in 10.0.0 (tsc TS2769). |
| 37 | (new) | Pointer to validators table in patterns | Add | Optional, nullable, literal, union, and transformation schemas are there. |
| 38 | 85 | `### Function registration` | Keep | Heading still applies. |
| 39 | 87 | `internalQuery/Mutation/Action` | Update | `FunctionSpec.internal*` in spec, `FunctionImpl.make` in impl, codegen registers; platform meaning kept verbatim. |
| 40 | 88 | `query/mutation/action` public | Update | `FunctionSpec.public*`; public-Internet rule and HTTP-commit rule kept. |
| 41 | 89 | Cannot register through `api`/`internal` | Update | Through `refs`; codegen registers; `GroupImpl.finalize` requires every impl. |
| 42 | 90 | ALWAYS include arg validators | Update | Confect validates every function; field-map `args`; omitted args still validated; `returns` required (verified). |
| 43 | (new) | Plain Convex functions need Convex validators | Add | Confect does not decode `convex*` functions (E12), so the upstream rule still applies there. |
| 44 | 92 | `### Function calling` | Keep | Heading still applies. |
| 45 | 94 | `ctx.runQuery` | Update | `QueryRunner` service; adds HTTP actions (pattern table Q, M, A, H). |
| 46 | 95 | `ctx.runMutation` | Update | `MutationRunner` service. |
| 47 | 96 | `ctx.runAction` | Update | `ActionRunner` service. |
| 48 | 97 | Call action from action only across runtimes | Update | "helper async function" to "Effect helper". |
| 49 | 98 | Few calls from actions | Keep | Platform rule, unchanged. |
| 50 | 99 | Pass a `FunctionReference` | Update | Pass a ref from `confect/_generated/refs`, not the impl. |
| 51 | 100 | Nested calls are subtransactions | Update | Kept; adds `Effect.catchTag` for declared errors and `Effect.catchDefect` for others. |
| 52 | 101 | `transactionLimits` third argument | Update | `ctx.runQuery` to `runQuery`; platform semantics verbatim. |
| 53 | 103-111 | `try/catch` `transactionLimits` code | Update | Replaced with E6 `postBounded` excerpt. |
| 54 | 113 | Supported `transactionLimits` fields | Keep | Platform fact, unchanged. |
| 55 | 115 | Same-file return type annotation | Remove | Confect refs carry the args, returns, and error schemas, so the circularity workaround does not apply (patterns E6). Replaced by row 57. |
| 56 | 117-132 | Same-file annotation code | Remove | Raw Convex API with no Confect counterpart; see row 55. |
| 57 | (new) | Refs carry schemas; no annotation needed | Add | Replacement for rows 55-56. |
| 58 | 134 | `### Function references` | Keep | Heading still applies. |
| 59 | 136 | `api` object | Update | `refs.public`. |
| 60 | 137 | `internal` object | Update | `refs.internal`. |
| 61 | 138 | File-based routing | Update | Routing from spec files; adds Convex function name `example:f`. |
| 62 | 139 | Private function ref | Update | `refs.internal.example.g`. |
| 63 | 140 | Nested directories | Update | `confect/messages/access.spec.ts`; fixes "can also registered" grammar. |
| 64 | (new) | Rename changes the API | Add | Patterns layout rule; file path is the API path. |
| 65 | 142 | `### Pagination` | Keep | Heading still applies. |
| 66 | 144 | Define pagination syntax | Update | `FunctionSpec.publicPaginatedQuery` with `item`; Confect adds arg and result schema. |
| 67 | 146-160 | `paginationOptsValidator` code | Update | Replaced with E2 spec excerpt `listMineByTag`. |
| 68 | (new) | Paginated impl excerpt | Add | E3 `listMineByTag` impl, second half of the split sample. |
| 69 | 162 | "Note: paginationOpts is an object..." | Keep | Unchanged. |
| 70 | 164 | `numItems` | Update | Validator `v.number()` to schema `Schema.Finite` (verified `PaginationOptions.ts`). |
| 71 | 165 | `cursor` | Update | Validator to `Schema.Union([Schema.String, Schema.Null])`. |
| 72 | 166 | `endCursor` | Keep | Unchanged. |
| 73 | 167 | `maximumRowsRead` | Keep | Unchanged. |
| 74 | 168 | `maximumBytesRead` | Keep | Unchanged. |
| 75 | 169 | `id` | Update | "accepted by `PaginationOptions.PaginationOptions`". |
| 76 | 171 | Always validate with `paginationOptsValidator`; pass unchanged | Update | Do not declare `paginationOpts` (redeclare fails typecheck, verified); pass unchanged rule kept verbatim. |
| 77 | 173 | Return object intro | Keep | Unchanged. |
| 78 | 175 | `page` | Keep | Unchanged. |
| 79 | 176 | `isDone` | Keep | Unchanged. |
| 80 | 177 | `continueCursor` | Keep | Unchanged. |
| 81 | 178 | `splitCursor` / `pageStatus` | Keep | Unchanged; matches `PaginationResult.ts`. |
| 82 | 180 | `paginationResultValidator` | Update | Confect builds `PaginationResult.PaginationResult(item)`. |
| 83 | 182 | `## Schema guidelines` | Keep | Heading still applies. |
| 84 | 184 | Define schema in `convex/schema.ts` | Update | One file per table in `confect/tables/`; codegen writes `convex/schema.ts`. |
| 85 | 185 | Import from `convex/server` | Update | `Table` from `@confect/core`, `Schema` from `effect/Schema`. |
| 86 | (new) | E1 `confect/tables/notes.ts` code | Add | Shows table file, `Id`, `optionalKey`, and named indexes. |
| 87 | 186 | System fields | Update | Schemas `Schema.Finite` and `Id(tableName)` (verified `SystemFields.ts`). |
| 88 | 187 | Include all index fields in the name | Keep | Same rule and same `.index(name, fields)` call. |
| 89 | 188 | Query index fields in order | Keep | Platform rule, unchanged. |
| 90 | 189 | No unbounded array fields | Update | Example `v.array(v.object(...))` to `Schema.Array(Schema.Struct(...))`. |
| 91 | 190 | Separate high-churn data | Keep | Platform rule, unchanged. |
| 92 | 192 | Staged index | Update | Platform fact kept; Confect 10.0.0 cannot declare it (no options argument; codegen owns schema); plan for blocking deploy. |
| 93 | 194 | `## Authentication guidelines` | Keep | Heading still applies. |
| 94 | 196 | `convex/auth.config.ts` | Update | Write `confect/auth.ts`; codegen copies it; missing file means `NoUserIdentityFoundError`. |
| 95 | 197 | "Example `convex/auth.config.ts`" | Update | File name `confect/auth.ts`. |
| 96 | 199-208 | Auth config code | Update | Replaced with E14 `confect/auth.ts` (same shape; comment and example domain). |
| 97 | 210 | `domain` / JWKS / `aud` paragraph | Keep | Platform fact, unchanged. |
| 98 | 212 | `ctx.auth.getUserIdentity()` | Update | `Auth` service; an Effect; fails with `NoUserIdentityFoundError` instead of `null`; field list kept. |
| 99 | (new) | E5 `store` excerpt | Add | Shows identity read and `tokenIdentifier` lookup. |
| 100 | 213 | Prefer `tokenIdentifier` | Keep | Platform rule, unchanged. |
| 101 | 214 | Never accept user id for authz | Update | Derive through `Auth` or middleware (E5). |
| 102 | 215 | Use `ConvexProviderWithAuth` | Update | Adds "Confect does not change client auth setup"; rest verbatim. |
| 103 | 217-229 | Client provider code | Keep | Client code; Confect does not change it. |
| 104 | 231 | `useAuth` return shape | Keep | Unchanged. |
| 105 | 233 | `## Typescript guidelines` | Keep | Heading still applies. |
| 106 | 235 | `Id` type from `dataModel` | Update | `GenericId.GenericId<"users">` type; `Id("users")` schema; generated `id` exports no type (version note). |
| 107 | 236 | `Doc<"t">` | Update | `UsersDoc` from `confect/_generated/docs` or `typeof users.Doc.Type`. |
| 108 | 237 | `QueryCtx`/`MutationCtx`/`ActionCtx` typing | Update | Handlers take no ctx; use services; raw ctx services only for escape hatches; never `any`. |
| 109 | 238 | `Record` with `Id` key | Update | `Schema.Record(Id, ...)` fails tsc in 10.0.0; use string key schema and type the value in code. |
| 110 | 240-258 | `Record<Id<"users">, string>` query code | Update | Replaced with compiled `directory.spec.ts` excerpt. |
| 111 | (new) | `namesById` impl excerpt | Add | Second half of the split sample (compiled and tested). |
| 112 | 260 | Strict id types | Update | `Id("users")` not `Schema.String`; `GenericId.GenericId<"users">` not `string`. |
| 113 | 261 | Typed app env via `defineApp({ env })` | Update | Not surfaced by Confect (patterns gap); use `Config`; platform `CONVEX_SITE_URL`/`CONVEX_CLOUD_URL` rule kept; `defineApp({ env })` in user-owned `convex/convex.config.ts` stays valid as an escape hatch. |
| 114 | (new) | E10 `describe` excerpt | Add | Shows `Config.String` and `Config.Int`. |
| 115 | 263 | `## Full text search guidelines` | Keep | Heading still applies. |
| 116 | 265 | FTS example intro | Update | Adds that channel and query are arguments. |
| 117 | 267-272 | Unfenced `withSearchIndex` code | Update | Replaced with E3 `search` excerpt in a fenced block. |
| 118 | 274 | `## Vector search guidelines` | Keep | Heading still applies. |
| 119 | 276 | Store embeddings in `v.array(v.float64())` | Update | `Schema.Array(Schema.Finite)`; table file. |
| 120 | 278-288 | `defineTable` vector index code | Update | Replaced with E1 `confect/tables/documents.ts`. |
| 121 | 290 | `dimensions` must match | Keep | Platform rule, unchanged. |
| 122 | 291 | `ctx.vectorSearch` only in actions | Update | `VectorSearch` service, actions only. |
| 123 | 293-299 | `ctx.vectorSearch` code | Update | Replaced with E9 `similar` excerpt (includes hydration). |
| 124 | 301 | Filter supports equality and `q.or` only | Keep | Platform rule, unchanged. |
| 125 | 302 | Returns `{ _id, _score }`; hydrate | Update | `ctx.db` to `DatabaseReader`. |
| 126 | 304 | `## Component guidelines` | Keep | Heading still applies. |
| 127 | 306 | Install and mount in `convex/convex.config.ts` | Update | Adds that you own the file and codegen does not overwrite it. |
| 128 | 308-315 | `defineApp` aggregate code | Keep | `convex/convex.config.ts` is unchanged under Confect. |
| 129 | 317 | `components` from `convex/_generated/api` | Update | `components` from `confect/_generated/components`. |
| 130 | (new) | E6 `rateLimits.ts` code | Add | Shows the Confect registry passed to a client class. |
| 131 | 318 | Component functions not exposed; authz first | Keep | Platform rule, unchanged. |
| 132 | 319 | Same-transaction mirror rule | Update | Kept; adds raw ctx service plus `Effect.promise`. |
| 133 | (new) | E6 `send` excerpt | Add | Shows raw ctx component call in the same mutation. |
| 134 | 320 | Authoring a local component | Update | Platform facts kept; path no longer under `convex/`, because codegen deletes non-group `.ts` files there (verified in `@confect/cli` `utils.mjs`); write it in plain Convex. |
| 135 | 321 | Use `@convex-dev/rate-limiter` | Keep | Unchanged. |
| 136 | 322 | Use `@convex-dev/agent` | Update | Kept verbatim; adds that `ctx` is the raw ctx service and `components` comes from Confect. |
| 137 | 323 | Use `@convex-dev/workpool` | Keep | Unchanged. |
| 138 | 324 | Use `@convex-dev/presence` | Keep | Unchanged. |
| 139 | 325 | Component mutation is a subtransaction | Update | Kept; adds `Effect.promise` defect and `Effect.catchDefect`. |
| 140 | 326 | `createFunctionHandle` | Update | Not wrapped; `Ref.getFunctionReference(refs.internal...)`; receiver uses raw ctx. |
| 141 | (new) | E12 `clearResultsHandle` excerpt | Add | Compiled form of the handle mint. |
| 142 | 328 | `## Query guidelines` | Keep | Heading still applies. |
| 143 | 330 | Prefer `.withIndex`; `.filter` after scan | Update | Every query names an index; no `.filter`; `Stream.filter` or `paginate` filter; rows-read fact kept. |
| 144 | 331 | No wall clock in queries | Update | Kept verbatim; adds 10.0.0 facts (`Date.now()` and `Clock.currentTimeMillis` read real time; `Effect.timed` reports zero). |
| 145 | 332 | Return bounded results | Keep | Same method names in Confect. |
| 146 | 333 | No `.collect().length`; counters; aggregate | Keep | Platform rule, unchanged. |
| 147 | 334 | No query `.delete()` | Update | `.stream()` instead of async iteration; `writer.table(...).delete`. |
| 148 | 335 | Batch, schedule continuation, metrics | Update | `scheduler.runAfter(Duration.zero, refs.internal...)`, `.stream()` with `Stream.runForEachWhile`, `TransactionMetadata.getMetrics()`. |
| 149 | (new) | `purgeCompletedBySize` excerpt | Add | Compiled and tested variable-size batch with `bytesRead.remaining`. |
| 150 | 336 | `.unique()` | Update | `get(indexName, ...values)`; `GetByIndexFailure` on none; defect on many (verified). |
| 151 | (new) | `findByToken` excerpt | Add | Compiled and tested. |
| 152 | 337 | Async iteration: no `collect`/`take` | Update | No `for await`; use `.stream()` and Stream operators. |
| 153 | 339 | `### Ordering` | Keep | Heading still applies. |
| 154 | 341 | Default ascending; table scan uses `by_creation_time` | Update | Confect has no plain scan; name `by_creation_time`. |
| 155 | 342 | `.order('asc'/'desc')` | Update | Last argument of `.index(...)`. |
| 156 | 343 | Index queries are ordered by index columns | Keep | Unchanged. |
| 157 | 344 | `_creationTime` tiebreak | Update | `.order("desc")` to a `"desc"` order. |
| 158 | 346 | `## Mutation guidelines` | Keep | Heading still applies. |
| 159 | 348 | `ctx.db.replace` | Update | `writer.table(t).replace`; missing document is a defect (verified by test). |
| 160 | (new) | `reset` excerpt | Add | Compiled and tested replace. |
| 161 | 349 | `ctx.db.patch` | Update | `writer.table(t).patch`; `GetByIdFailure`; `undefined` removes; patch reads first (verified `DatabaseWriter.ts`). |
| 162 | (new) | `markCompleted` excerpt | Add | Compiled and tested patch with typed recovery. |
| 163 | 351 | `## Action guidelines` | Keep | Heading still applies. |
| 164 | 353 | `"use node"` for Node built-ins | Update | Node group via `GroupSpec.makeNode()`; codegen adds the directive. |
| 165 | 354 | Never mix `"use node"` with queries or mutations | Update | Node group accepts only Node actions (verified `GroupSpec.ts`); platform rule kept. |
| 166 | 355 | `fetch()` needs no Node | Update | "no Node group" instead of `"use node"`. |
| 167 | 356 | No `ctx.db` in actions | Update | No `DatabaseReader`/`DatabaseWriter`; use runners. |
| 168 | 357 | Action example intro | Keep | Unchanged. |
| 169 | 359-369 | `action({...})` code | Update | Replaced with `jobs.spec.ts` excerpt; `console.log` to `Effect.log`. |
| 170 | (new) | `exampleAction` impl excerpt | Add | Second half of the split sample. |
| 171 | 371 | `## Scheduling guidelines` | Keep | Heading still applies. |
| 172 | 373 | `### Cron guidelines` | Keep | Heading still applies. |
| 173 | 375 | Only `crons.interval` / `crons.cron` | Update | `CronJob.make` with `Cron` or `Duration` (maps to those two only, verified `CronJobs.ts`); no hourly/daily/weekly. |
| 174 | (new) | Cron granularity, whole-unit durations, UTC | Add | Patterns E15 platform facts for the schedule types. |
| 175 | 376 | Pass a `FunctionReference` | Update | Pass a ref from `refs`. |
| 176 | 377 | Define the `crons` object | Update | `confect/crons.ts` default-exports `CronJobs.make()`; codegen writes `convex/crons.ts`. |
| 177 | 379-397 | `cronJobs()` code | Update | Replaced with compiled `confect/crons.ts`; the two-hour interval job is kept as `Duration.hours(2)`. |
| 178 | 399 | Register functions in `crons.ts` | Update | Inverted: `confect/crons.ts` holds only jobs; declare targets in a group spec. |
| 179 | 400 | Import `internal` for crons | Update | Use `refs.internal`. |
| 180 | 402 | `## Testing guidelines` | Keep | Heading still applies. |
| 181 | 404 | `convex-test` + vitest + edge-runtime | Update | Adds `@confect/test` and `@effect/vitest`; same-version rule; inline `convex-test`; exclude `*.spec.ts`; latest-versions rule kept for `convex-test`, `vitest`, `@edge-runtime/vm`; `edge-runtime` kept as the direct instruction; examples verified with `node` only (stated in text). |
| 182 | 406 | Test files inside `convex/`; module map | Update | Tests go outside `convex/` (codegen deletes them); `TestConfect.layer` with module glob. |
| 183 | 408-423 | `convexTest` code | Update | Replaced with E16 `test/TestConfect.ts`. |
| 184 | (new) | E16 action test excerpt | Add | Shows a test that uses the layer. |
| 185 | 425 | `modules` and vite/client paragraph | Keep | Still true for `TestConfect.layer`. |
| 186 | 427 | vite/client only in test files that glob | Update | "test files" to "files": the helper, not the test, calls the glob. |
| 187 | 428 | No `types` allowlist for missing packages | Keep | Unchanged. |
| 188 | (new) | Register called components with `convex-test` | Add | Patterns E16 test note. |
| 189 | (new) | `TestConfect` helper methods | Add | Patterns concept map. |
| 190 | 430 | `## File storage guidelines` | Keep | Heading still applies. |
| 191 | 432 | `ctx.storage.getUrl()` returns `null` | Update | `storage.getUrl` returns a `URL` and fails with `BlobNotFoundError`; return with `URLFromString`. |
| 192 | 433 | Do not use `getMetadata` | Update | Kept; adds that the Confect `Storage` service does not expose it (verified). |
| 193 | 435 | Query `_storage` with `ctx.db.system.get` | Update | `reader.table("_storage").get(id)`; `FileMetadata` fields kept as text. |
| 194 | 437-457 | `ctx.db.system.get` code | Update | Replaced with E8 `metadata` spec excerpt. |
| 195 | (new) | E8 `metadata` impl excerpt | Add | Second half of the split sample. |
| 196 | 459 | Convert to and from `Blob` | Update | Kept; adds that `store` and `get` work only in actions and HTTP actions. |
