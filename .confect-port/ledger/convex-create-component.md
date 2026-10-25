# Ledger: convex-create-component

Port target: Confect 10.0.0. Directory name is unchanged (a later step renames it).
Key finding: Confect does not build components (patterns file, Gaps table; verified in
@confect/cli dist: codegen only reads installed components and writes
`confect/_generated/components.ts`, typed from `<component dir>/_generated/component.js`).
So component code stays plain Convex. App wrappers become Confect specs and impls.
Not "platform only": this skill has code on both sides of the boundary.

Actions: keep, update, add, remove. Line numbers refer to the original file.

## SKILL.md (original 325 lines)

| Source | Action | Reason |
| --- | --- | --- |
| 1-7 frontmatter name | update | `confect-create-component`. |
| 1-7 frontmatter description | update | Triggers on Confect projects (confect/ directory). |
| 9-12 title and intro | update | Title is Confect form. Intro kept. |
| (new, after intro) | add | Explains: Confect mounts and calls components, does not build them. Points to confect skill `references/confect-patterns.md` and to `convex/_generated/ai/guidelines.md` in the app (the confect skill has no `references/guidelines.md`). Says not to run `npx convex ai-files install` (new text; the upstream file has no such pointer). |
| 14-20 When to Use | keep, update 1 bullet | "existing Confect app". |
| 22-27 When Not to Use | keep, update 1 bullet | App orchestration stays in `confect/` (was `convex/`). |
| 29-34 steps 1-4 | keep | Platform and process rules. Step 4 bullet "wrappers" now says "Confect wrappers". |
| 35-36 step 5 | update | Adds: write plain Convex in component, no `@confect/*` (Confect codegen does not read component code; Effect itself is not banned). |
| 37-38 step 6 | keep | Component own `./_generated/server` rule is platform. |
| 39-40 step 7 | keep | `app.use(...)` and `convex/convex.config.ts` are unchanged (user-owned file). |
| 41-42 step 8 | update | `components` from `confect/_generated/components`. Raw ctx services plus `Effect.promise`. Confect does not wrap component calls (E6, Gaps). |
| 43-44 step 9 | update | Wrappers are Confect spec plus impl. |
| 45-46 step 10 | update | Points to validation steps (Confect codegen, tsc). |
| 56-67 Choose the Shape table | keep | Process only. |
| 69-76 Default Approach | keep, update 2 bullets | Notes user owns convex.config.ts. Component codegen replaces "npx convex dev generates", because Confect codegen needs `_generated/component.js` first. |
| 78-80 Skeleton intro | update | Says which side is plain Convex and which is Confect. |
| 82-87 component convex.config.ts | keep | Plain Convex. |
| 89-101 component schema.ts | keep, update comment | Marked plain Convex. |
| 103-140 component lib.ts | keep, update comment | Marked plain Convex. `v.*`, `ctx.db` stay: Confect does not apply inside a component. |
| 142-151 app convex.config.ts | keep | No change in Confect (patterns: "You own that file"). |
| 153-186 app wrapper `mutation`/`query`, `v.*`, `getAuthUserId`, `ctx.runMutation` | update | Replaced by `confect/notifications.spec.ts` and `.impl.ts`. `FunctionSpec.publicMutation/publicQuery`, `Schema.*`, `Auth` with `tokenIdentifier` (E5, replaces `@convex-dev/auth` `getAuthUserId`), `MutationCtx`/`QueryCtx` with `Effect.promise`, `components` from `confect/_generated/components`. Typed error `Unauthenticated` replaces `throw new Error("Not authenticated")`. Platform rule kept: auth check in the app, then component call. |
| (new, after 186 note) | add | Notes: component write is in the caller's transaction; client name of the wrapper. |
| 188-190 reference path note | keep | Platform fact. |
| 192-194 rule: auth in app | update | Adds the `Auth` service and `tokenIdentifier`. |
| 195-196 rule: env in app | update | Adds `Config`. |
| 197-199 rule: parent IDs as strings | keep | Platform fact. |
| 200-201 rule: no `v.id("parentTable")` | update | Adds Confect `Id(...)` is also not valid there. |
| 202-203 rule: own `_generated/server` | update | Adds: no `confect/` or `@confect/*` imports in component code. |
| 204-206 rule: no direct client exposure | update | Wrappers are Confect specs and impls. |
| 207-208 rule: HTTP routes in app | update | `confect/http.ts` plus `httpActionGeneric` escape hatch (E13). `convex/http.ts` is generated. |
| 209-211 rule: pagination | keep, add note | `paginator` rule is platform. Notes it applies to component code. |
| 212-213 rule: indexes not `.filter()` | keep, add note | Confect query builder has no `.filter()` (Gaps). |
| 214-215 rule: args and returns validators | update | `v.*` in component, Effect Schema in wrapper spec. |
| (new rule) | add | Component names: letters, digits, underscore, first character a letter or underscore (regex `^[A-Za-z_][A-Za-z0-9_]*$`); unique `name` for duplicates. Verified in ConvexConfig.mjs (`InvalidConvexConfigError`). |
| 217 Patterns heading | keep | |
| 219-225 Bad: ctx.auth and process.env | keep, add comment | Still wrong inside a component. Comment says it is component code. |
| 227-237 Good: auth and env in app | update | `Auth`, `Config.String`, `ActionCtx`, `Effect.promise`. Snippet now has imports, `({ text })`, and `Effect.orDie` so `ConfigError` stays out of the spec error channel. Error class renamed `Unauthenticated` (was `NotSignedIn`) in all files. |
| 239-244 Bad: `components.notifications.send` | keep, add comment | Same anti-pattern. Comment explains clients cannot call it. |
| 246-262 Good: wrapper mutation | update | Confect `FunctionImpl.make` form. |
| 264-271 Bad: `v.id("users")` | keep, add comment | Component-side plain Convex. |
| 273-278 Good: `v.string()` | keep, add | Adds the wrapper-spec form `Schema.String`. |
| 280-284 Advanced Patterns pointer | keep, add 1 sentence | Says which side is plain and which is Confect. |
| 286-300 Validation list | update | Component codegen, `npx confect codegen`, `tsc --noEmit`, `npx convex dev` with `npx confect dev`. Kept the app-level `npx convex codegen` as step 3, because Confect codegen does not write `convex/_generated/`. |
| 291-300 Important bullets | keep, add | Keeps CONVEX_DEPLOYMENT and login bullets. Adds ordering reason, tsc gate, no edits to `_generated`. |
| 302-311 Reference Files, official docs link | keep | Platform. |
| 313-325 Checklist | keep, update, add | Adds plain-Convex component, Confect wrapper, auth-before-call, codegen and tsc rows. |

Counts (SKILL.md): kept 19, updated 22, added 7, removed 0.

## references/advanced-patterns.md (original 146 lines)

| Source | Action | Reason |
| --- | --- | --- |
| 1-4 intro | update | Says which side is plain Convex. |
| 6-10 Function Handles intro | keep | |
| 12-24 App side `mutation`, `internal.myModule.processItem`, `createFunctionHandle`, `ctx.runMutation` | update | Replaced by plain Convex `processItem` (`internalMutationGeneric`), spec (`FunctionSpec.convexInternalMutation`, type-only import), impl with `Ref.getFunctionReference(refs.internal.jobs.processItem)` in `Effect.promise` (E12). Confect does not wrap handles (Gaps). |
| 26-39 Component side handle | keep, add comment | Plain Convex. |
| 41-44 Deriving validators intro | keep, add note | Confect tables do not give component validators. |
| 46-68 schema.doc sample | keep, fix | Plain Convex. Fixed `handler: async (ctx)` to `(ctx, args)` because the body uses `args` (upstream bug). Not otherwise verified (`v.nullable`, `users` table lookup left as upstream). |
| 70-101 globals table and `configure` | keep, add comments | Plain Convex in component. |
| 103-128 class-based client | keep, add note | Plain TypeScript. Takes raw ctx with `runMutation`. |
| 130-146 App usage with `mutation`, `getAuthUserId`, `components` from `_generated/api` | update | Confect `FunctionImpl.make`, `Auth`, `MutationCtx`, `components` from `confect/_generated/components`. |

Counts: kept 5, updated 4, added 4, removed 0.

## references/local-components.md (original 39 lines)

| Source | Action | Reason |
| --- | --- | --- |
| 1 title | update | Confect context. |
| 3-4 intro, 6-10 When to Choose | keep | |
| 12-24 layout | keep | Component stays under `convex/components/<name>/`. |
| 28-29 workflow bullets 1-2 | keep | |
| 30-32 workflow bullets 3-5 | keep, extend | Wrappers are Confect specs and impls. |
| (new bullets) | add | Plain Convex component. Codegen order. Do not place component in `confect/`. |
| 34-39 checklist | keep, add 2 rows | Wrappers are Confect. confect codegen and tsc pass. |

Counts: kept 8, updated 3, added 4, removed 0.

## references/packaged-components.md (original 54 lines)

| Source | Action | Reason |
| --- | --- | --- |
| 1-4 title and intro | update (title) | |
| 6-10 When to Choose | keep | |
| 12-17 Default Approach | keep | Platform. |
| 19-30 Build Flow | keep | Platform commands. |
| (new paragraph after 30) | add | Confect installs the package, registry typed from `ComponentApi`. |
| 32-39 Package Exports | keep | Platform. |
| 41-46 Testing | update | Component tests stay `convex-test`. Example app is a Confect app. `TestConfect.layer` does not expose the convex-test instance (checked in `@confect/test` d.ts), so a Confect wrapper test cannot register the component. |
| 48-54 Checklist | update 1 row | Example app is a Confect app. |

Counts: kept 6, updated 3, added 2, removed 0.

## references/hybrid-components.md (original 38 lines)

| Source | Action | Reason |
| --- | --- | --- |
| 1 title | update | |
| 3-14 What This Means, bullets | keep, add notes | Plain Convex plus Confect statement. Shared code must not import `@confect/*` if component code uses it. |
| 16-38 Default Advice, Risks, Checklist | keep | Process only, platform only. |

Counts: kept 4, updated 3, added 0, removed 0.

## agents/openai.yaml (14 lines)

| Source | Action | Reason |
| --- | --- | --- |
| display_name, short_description, default_prompt | update | Confect names and wording. |
| icon paths, brand_color, policy | keep | |

Counts: kept 3, updated 3, added 0, removed 0.

## assets/icon.svg: keep, no change (no Convex text).

## Open questions

1. `TestConfect.layer` cannot register a component. Is there a supported way to test a Confect wrapper that calls a component? Docs may differ for newer versions.
2. Local component directory under `convex/components/`: codegen does not delete it (no removal code found for that path), but `convex/` is described as generated. Confirm with a real run.
3. Code samples here were not compiled (no installs allowed, no project). The patterns used are copied from the verified E2, E5, E6, E12 blocks. Compile once in a sample project.
4. `FunctionSpec.convexInternalMutation` as the target of a function handle that a component calls: the patterns file shows only the spec form. The handle path was not run.
5. Upstream sample uses `v.nullable` and a `users` lookup in a component. Left unchanged.
6. Other adapted skills (convex-add, convex-advisor, convex-auth, and others) also point to a missing confect `references/guidelines.md`. Fix them together in one step.
