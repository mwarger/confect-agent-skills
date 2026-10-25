---
name: confect-quickstart
description: "Get a barebones Confect (Effect on Convex) + web template running from a one-sentence idea. Use when the user starts a new Confect or Convex app, or adds Confect to an existing frontend."
---

<!-- GENERATED from convex-agents content/capabilities/quickstart.json — do not edit by hand. -->
<!-- Adapted to Confect. Confect changes how code is written. It does not change how Convex behaves. -->

# Quickstart: a barebones Confect template, running

Stand up a barebones Next.js + Convex template from the idea, locally, with an anonymous dev deployment. Then add Confect on top of it. Minimal by design: local dev servers, no publish step, no pre-baked auth.

Confect is Effect on Convex. Read `../confect/references/confect-patterns.md` for the Confect form of every Convex pattern. For platform rules, read `convex/_generated/ai/guidelines.md` in the project when it exists. The Convex guidelines stay the source for platform rules.

## Workflow

1. Run recipe `quickstart-recipe@^2` with {idea, template} (the pack fetches + caches it; pinned offline fallback). It creates the project, installs deps, starts the backend (anonymous) and the web dev server. The recipe makes a plain Convex project.
2. Keep the recipe's backend running. Add Confect to the project (ask first when the user did not request it). Confect needs Node 24 or later. Install these with the project's package manager:
   - Dependencies: `@confect/core`, `@confect/server`, `@confect/cli`, `effect` (v4) and `@effect/platform-node` (v4).
   - Dev dependencies: `@confect/test` and `convex-test`.
   - `convex` must be 1.45.0 or later. Upgrade it when the template has an older version.
3. Move the template code into `confect/`. Keep `confect/` next to `convex/`. Put one file per table in `confect/tables/`. Put function specs and impls in `confect/<group>.spec.ts` and `confect/<group>.impl.ts`. `confect codegen` overwrites `convex/schema.ts`. It writes the schema from `confect/tables/`. A template table that is not in `confect/tables/` is dropped. So do this for the template's existing `convex/schema.ts` and `convex/*.ts` functions:
   - Move each table to a file in `confect/tables/`.
   - Move each function to a spec and an impl. Or keep it as a plain Convex function in `confect/<group>.ts` (section E12 in `confect-patterns.md`).
   - Delete the old template function files from `convex/`. Do this after the moved code works.
   - Update each frontend call that uses `api.*` from `convex/_generated/api` to use the new function path.
4. Run `npx confect codegen`. It writes `confect/_generated/` and the generated files in `convex/`. Do not edit those files. During development, keep the recipe's backend running. Add `npx confect dev` in a second terminal.
5. When the recipe prints the dev URL, open it for the user.
6. Present a short plan and CONFIRM before building features beyond the template.

## Rules

- Never re-run the recipe if it already reported success.
- Delegate any code under `confect/` (and any plain Convex code under `convex/`) to the `confect-expert` capability.
- Don't add Postgres/Redis/Express — use Convex primitives.
- Don't add hosting/publish or pre-baked auth here — keep the template minimal unless the user asks for more.
- DEGRADATION RULE — if the scaffold cannot run (non-interactive session, no network, a sandboxed temp dir, or the user just wants code, not an app): skip the recipe and write a standard Confect project directly. First create `convex/`: run `npx convex dev --once` (see the SELF-VERIFY RULE for which deployment), or write `convex/tsconfig.json` by hand. Then install the Confect packages (see step 2). `npx confect codegen` fails with `ConvexDirectoryNotFoundError` when `convex/` does not exist. ALL backend code goes under `confect/` (tables in `confect/tables/`, specs and impls in `confect/<group>.spec.ts` and `confect/<group>.impl.ts`) — NEVER at the project root. Convex functions only run from the `convex/` directory, and `npx confect codegen` writes them there. Do not edit the generated files in `convex/` or `confect/_generated/`. You own `convex/convex.config.ts` and `convex/tsconfig.json`. Write ZERO scaffold/documentation files (no START_HERE.md, ARCHITECTURE.md, MANIFEST.txt, README walls) unless explicitly asked. "Build me a backend" means code, not ceremony.
- Data access + imports — before writing any confect/*.ts:
  - Never an unbounded `.collect()` on a table that can grow. Get the reader with `const reader = yield* DatabaseReader`. Use `reader.table(name).index(...)` with `.paginate(paginationOpts)` or `.take(n)`. Declare a paginated function with `FunctionSpec.publicPaginatedQuery` or `FunctionSpec.internalPaginatedQuery`. Confect adds the `paginationOpts` arg.
  - Every Confect query names an index. Use an index, not a filter, for anything that would be a SQL WHERE. The query builder has no `.filter()`. Only `.paginate` takes an optional second argument, a filter callback. It still reads every row that it scans. For the rest, use `.stream()` with `Stream.filter` (the rows read do not go down).
  - The index range callback in `.index(name, (q) => ...)` only has `eq`/`gt`/`gte`/`lt`/`lte` — there is no `.range(...)` method.
  - Imports: `FunctionSpec` and `GroupSpec` come from `@confect/core`. `FunctionImpl` and `GroupImpl` come from `@confect/server`. Services (`DatabaseReader`, `DatabaseWriter`, `QueryRunner`, `MutationRunner`, `ActionRunner`, `Scheduler`, `Auth`, `Storage`) come from `./_generated/services`. `refs` comes from `./_generated/refs`. Use `FunctionSpec.publicQuery`/`publicMutation`/`publicAction` and `FunctionSpec.internalQuery`/`internalMutation`/`internalAction`. NEVER import `query`/`mutation`/`action` from `convex/_generated/server`, or `api`/`internal` from `convex/_generated/api`, in Confect code.
  - A `*.spec.ts` file must not value-import `@confect/server`. Use `import type`.
  - Use `convex/server` and `convex/values` only in plain Convex functions (the escape hatch in `confect-patterns.md`, section E12).
  - Use `Schema.Literal("exact value")` (or `Schema.Literals([...])` for a set) for fixed string/enum members, not a bare `Schema.String`. Use `Schema.Finite` for numbers, not `Schema.Number`.
  - Codegen writes `"use node"` for Node action groups. Never write it by hand in a spec or impl file. Declare Node action groups with `GroupSpec.makeNode()` and `FunctionSpec.publicNodeAction` or `internalNodeAction`. Never put a Node-runtime action in a group that also has a query or mutation. Never import a Node builtin (`crypto`/`fs`/`path`/`http`/`child_process`/`os`, with or without the `node:` prefix) into a Convex-runtime group — including HTTP route handlers; use Web Crypto (`crypto.subtle`) instead of `import`ing `crypto` where possible.
- Reserved names — never name a function with a JS reserved word (e.g. `delete`, `new`, `class`, `function`, `return`). Use `remove`, `create`, and so on. Never name a function `schema`, `http` or `crons`. Confect throws an error when it builds the spec for these names. A function name must match `/^[a-zA-Z_$][a-zA-Z0-9_$]*$/`. A table file name must start with a letter. It can only have letters, digits and `_`. It must not be a JS reserved word. So no `$`, and no leading `_` (e.g. a `confect/tables/_migrations.ts` file). `confect codegen` fails with `InvalidTableFilenameError` for a bad table file name. The file name is the table name. Convex also reserves a leading `_` on index names. It errors at push as `IndexNameReserved`.
- HTTP routes — `HttpRouter.make` in `confect/http.ts` registers one catch-all HTTP action. The Effect router owns the paths. Declare path parameters with the endpoint path (`HttpApiEndpoint.get("get", "/users/:id", { params: ... })`) and decode them with the `params` schema. For a plain Convex route added with `http.route({...})` on the returned router, there are no Express-style `:param` segments (`path: "/users/:id"` only matches that literal string and is dead code); use `pathPrefix` and parse the trailing segment yourself. Every plain `http.route({...})` `handler:` must be wrapped in `httpActionGeneric(...)` from `convex/server` — a bare `async (ctx, request) => {...}` type-checks but isn't a valid HTTP action. Handlers on the Effect router need no wrapper. Commit effects through an internal mutation.
- Calling functions — `runQuery`/`runMutation`/`runAction` (from the `QueryRunner`, `MutationRunner`, `ActionRunner` services) need a generated ref (`refs.public.foo.bar`/`refs.internal.foo.bar`), never a raw imported module member (`import * as queries from "./queries"; runQuery(queries.getX, ...)` fails). Confect refs carry the args, returns and error schemas. Plain Convex needs an explicit return type annotation when a function calls a function in the same file. Confect code does not need it.
- SELF-VERIFY RULE — before declaring backend work done, verify it generates, compiles and pushes: run `npx confect codegen`, then `npx tsc --noEmit` (codegen success alone is not enough), then push it to a deployment. Prefer the project's existing one; otherwise `npx convex dev --once` when `npx convex whoami` succeeds, and `CONVEX_AGENT_MODE=anonymous npx convex dev --once` ONLY when it does not. Forcing anonymous on a signed-in user rebinds `.env.local` and costs them the persistent, publishable cloud deployment they expect. Fix every error it reports before finishing — one verify round catches the wrong-relative-import / duplicate-symbol / unbalanced-paren class that otherwise breaks the deploy. `GroupImpl.finalize` fails to typecheck until every spec function has an impl.
