---
name: confect-explain-app
description: "Explain an existing Confect app (Effect on Convex, a confect/ directory) — data model + relationships, public vs internal functions, auth/ownership model, components, a request→data flow — read from the table files and the function specs. Read-only."
---

<!-- GENERATED from convex-agents content/capabilities/explain-app.json — do not edit by hand. -->

# Explain this Confect app

Before you can safely change an app you have to know what it is — and reading 15 function files top-to-bottom is slow and error-prone. This capability produces the map fast and accurately by reading the two sources that can't lie: the table files (the data model) and the function specs (`*.spec.ts` group specs, the declared queries/mutations/actions). It is deliberately DESCRIPTIVE — it explains what IS, hands judgment to the audit capabilities and changes to the fixers. It is also the natural first step of an optimize or self-heal session, and the reusable 're-explain the current architecture' that 'change what you built' depends on.

Before you read a Confect project, read `confect/references/confect-patterns.md` (the confect skill). The platform rules (limits, indexes, transactions) are the same as in plain Convex. Do not read `confect/_generated/` or the generated files in `convex/` as the source. Codegen writes them from the table files and specs.

## Workflow

1. DETECT the app: the `confect/` directory (`tables/`, `*.spec.ts`, `*.impl.ts`), the `convex/` directory (only `convex.config.ts` and `tsconfig.json` are hand-written), and whether a deployment exists (if one does, `functionSpec`/`tables` via the official MCP give the authoritative live surface; if not, read the source directly). Function names in the live surface follow the group paths: `confect/notes.spec.ts` gives `notes:<name>`. deploy-guard classifies any deployment read as read-only.
2. DATA MODEL: from `confect/tables/`, list every table (one file per table; the file name is the table name) with its fields and, crucially, its RELATIONSHIPS — which `Id("other")` fields point where, and which indexes exist (`.index(...)`, `.searchIndex(...)`, `.vectorIndex(...)` on the table builder; indexes reveal the intended access paths). Note `GenericId.GenericId("_storage")` fields: they point at file storage. Draw the foreign-key graph in words: 'tasks belong to projects (projectId) and users (ownerId); messages belong to conversations'.
3. FUNCTION SURFACE: enumerate every function in the group specs (`confect/<group>.spec.ts`), split PUBLIC (`FunctionSpec.publicQuery` / `publicMutation` / `publicAction` / `publicPaginatedQuery` / `publicNodeAction` — the attack/API surface) from INTERNAL (`FunctionSpec.internalQuery` / ... — not client-reachable), and for each give a one-line 'what it does + what it touches' (read the matching `*.impl.ts`). Include plain Convex functions (`FunctionSpec.convexPublicMutation` and the other `convex*` constructors) and say that they bypass Confect's Schema decoding. The public/internal split is the single most important thing a newcomer needs and the thing source-skimming most often gets wrong. Also list each function's declared `error` schema: it shows which failures the caller can tell apart.
4. AUTH / OWNERSHIP MODEL: state how identity is established (`confect/auth.ts` provider, which codegen copies to `convex/auth.config.ts`? a users table keyed by tokenIdentifier?) and how ownership is enforced (is there middleware, a `MiddlewareSpec` that provides the current user? It attaches to a group with `GroupSpec.make().middleware(...)`, or to one function with `FunctionSpec.publicQuery({...}).middleware(Spec)`. Check both places. is there a check on the `Auth` service inside the impl? which field is the owner?). Say plainly if there is NO auth foundation — that is load-bearing context for anyone about to change the app. Middleware does not cover plain Convex functions or subgroups: say which functions that leaves without a guard. (Describe the model; do not audit it for holes — that's confect-authz.)
5. COMPONENTS + EXTERNAL EDGES: list the `@convex-dev/*` components installed (`convex/convex.config.ts`) and what they provide, the HTTP routes (`confect/http.ts`) and crons (`confect/crons.ts`), and any external calls in actions (which APIs, which env vars, read with `Config`). Confect registers one catch-all HTTP action under `/`, so the live `functionSpec` shows one route. The Effect `HttpApi` groups hold the real paths. Read those groups to list the routes.
6. FLOW: trace 1-2 representative end-to-end paths ('client calls `notes.create` → middleware resolves the current user → impl inserts into notes scoped to that user with `DatabaseWriter` → `notes.listMine` reads it back by the by_authorId index') so the reader sees the moving parts connected, not just catalogued.
7. PRESENT as a scannable map (data model → public/internal functions → auth model → components/edges → a flow or two), accurate to the source. End by pointing at the next verbs: confect-reviewer/confect-authz to audit it, launch-readiness to score it, design/confect-expert to extend it. Never invent behavior the source doesn't show; if something is ambiguous, say so rather than guessing.

## Rules

- Read the table files + function specs (and the impls, and `functionSpec` when a deployment exists) as the source of truth — never describe behavior the code doesn't show; flag ambiguity instead of guessing.
- Do not treat generated files (`confect/_generated/`, generated `convex/` modules) as authored code. Describe the authored source.
- Lead with the two things a newcomer most needs and skimming most often gets wrong: the data-model relationship graph and the public-vs-internal function split.
- State the auth/ownership model plainly, including 'there is no auth foundation' when that's the case — but DESCRIBE it; auditing it for holes is confect-authz's job.
- Descriptive, not evaluative: explain-app maps what IS and hands judgment to the audit capabilities and changes to the fixers.
- Read-only: any deployment introspection is read-only (deploy-guard); the app is not modified.
- End by pointing at the right next verb (audit → reviewer/authz, score → launch-readiness, extend → design/expert).
