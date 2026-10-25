# Ledger: convex-auth (SKILL.md, 29 lines)

Source file: skills/convex-auth/SKILL.md. Directory name kept; a later step renames it.
Code content: API mentions and a shell command. The upstream default is `@convex-dev/auth`. The adaptation adds a verified path (external JWT provider) and keeps the upstream steps as Path B.

Verified in the `@confect/cli` 10.0.0 source: (1) codegen removes any `convex/*.ts` module that is not reserved and not a group in the spec; (2) `convex/schema.ts` comes only from `confect/tables/`; (3) `confect/auth.ts` is copied to `convex/auth.config.ts`.

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter `name: convex-auth` | update | `confect-auth`. |
| Frontmatter `description` | update | Confect app: `confect/auth.ts`, users table, RequireUser middleware, client sign-in. |
| GENERATED comment | keep | Unchanged upstream line. |
| (new) "Adapted to Confect" comment | add | Same marker as the adapted skills. |
| `# Add sign-in to the app` | keep | Unchanged. |
| Intro: install and wire @convex-dev/auth, provider, config, hooks, UI | update | Reworded to the Confect app. The "auth config is the #1 footgun" point kept. States `confect/auth.ts` is copied to `convex/auth.config.ts`. |
| (new) Pointer to E5, E14, guidelines | add | Task rule for references. |
| (new) `## Choose the path` | add | Path A (external JWT provider, verified in patterns E14, E5). Path B (`@convex-dev/auth`) is not verified because codegen deletes `convex/auth.ts` and owns the schema. The agent must ask the user before Path B. |
| Workflow 1: install @convex-dev/auth, add to convex.config.ts, pnpm jose | update | Moved to Path B step 1. `convex.config.ts` is user-owned, so the step stays valid. |
| Workflow 2: provider in `convex/auth.ts` | update | Path B step 2. The file path is dropped: a hand-written `convex/auth.ts` is removed by codegen. |
| Workflow 3: headless key generation with `jose`, node one-liner, envSet, NAME=VALUE, SITE_URL, delete .auth-keys.json | keep | Path B step 3. Platform facts kept word for word. |
| Workflow 4: write `convex/auth.config.ts` | update | Now `confect/auth.ts` (Path A step 2, Path B step 4). Never edit the generated file. |
| Workflow 5: client wiring, ConvexAuthProvider, shadcn add | update | Path A step 5 uses `ConvexProviderWithAuth` (patterns E14). Path B keeps `ConvexAuthProvider`. shadcn rule kept. |
| Workflow 6: verify sign-in round trip | update | Path A step 6 adds `users.store`, the signed-out `Unauthenticated` check, codegen and tsc. |
| (new) Path A steps 1, 3, 4 | add | Provider choice, codegen and env vars, users table plus `store` plus `RequireUser` middleware (E1, E5). |
| (new) `confect/auth.ts` code block | add | Copied from patterns E14, neutral domain. |
| (new) Bad and Good code blocks | add | Hand-written `convex/auth.config.ts` and user id argument are Bad. `Auth` service is Good (E5). |
| Rule: generate JWT keys with `jose`, no wizard, envSet or NAME=VALUE | keep | Kept word for word. Marked "Path B only". |
| Rule: always write auth.config.ts, else silently signed out | update | Now `confect/auth.ts`. Added `NoUserIdentityFoundError` consequence (E14). |
| Rule: passkeys by default | update | Marked Path B. Path A has no passkey default because the provider owns the method. |
| Rule: install shadcn primitives up front | keep | Unchanged. |
| Rule: verify a real sign-in | keep | Unchanged. |
| (new) Rule: never edit `convex/auth.config.ts` | add | Patterns table row for `convex/auth.config.ts`. |
| (new) Rule: identity from `Auth` or middleware; `tokenIdentifier`; no user id args | add | Patterns E5 and "Platform rules: Auth". |
| (new) Rule: `ConvexProviderWithAuth` on the client | add | Patterns E14 closing paragraph. |
| (new) Rule: no Path B without the user | add | Codegen limits above. |

## Review fixes

- Pointers use the full path. Codegen wording is now: writes `convex/auth.config.ts`, which re-exports `confect/auth.ts` (verified in `@confect/cli` `templates.mjs`). The two-Bad block is split into two blocks. The `ConvexProviderWithAuth` rule is scoped to Path A; Path B uses `ConvexAuthProvider`. Path B step 2 now tells the agent to stop and report the step as blocked.

## Open questions

- Path B is unverified. Options to test later: define the `authTables` shapes as Confect tables; keep the library functions as plain Convex functions in a group with `FunctionSpec.convex*` (E12). The name clash between `confect/auth.ts` (config) and a plain-function file named `auth.ts` is untested. `@convex-dev/auth` needs the exact module name `auth`.
- Decide whether Path B stays in the skill or is removed after a test. It was kept so no upstream platform fact is lost.
- `confect/auth.ts` copy wording is still old in `skills/convex-domains/SKILL.md`, `skills/convex-explain-app/SKILL.md` and `skills/confect/references/confect-patterns.md` (E14). Those files are outside this fix. Update them in the same way.
- The six skills have `name: confect-*` but the directory is still `convex-*`. The cross-references to confect-reviewer, confect-authz and confect-expert do not resolve yet. The later rename step must fix both together. Not changed here on purpose.
