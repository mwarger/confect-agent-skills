# Ledger: convex-add (SKILL.md, 26 lines)

Source file: skills/convex-add/SKILL.md. Directory name kept; a later step renames it.
Code content: API mentions only (component wiring, http, crons). Mostly platform and process text.

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter `name: convex-add` | update | Confect form: `confect-add`. |
| Frontmatter `description` | update | Triggers on an app with a `confect/` directory. Says the catalog procedure is translated to Confect. Catalog and component search kept. |
| GENERATED comment | keep | Unchanged upstream line, keeps later syncs small. |
| (new) "Adapted to Confect" comment | add | Same marker as the already adapted skills. |
| `# add` | keep | Unchanged. |
| Intro paragraph: "existing Convex app" | update | "existing Confect app". |
| Intro paragraph: catalog fetch, /capability/<id>.md, fallback, hosting, add-component | keep | Platform and process text. Unchanged. |
| (new) Paragraph: catalog and README code is plain Convex; translate; pointers to patterns and guidelines | add | Task rule: Confect changes how code is written. Points to `references/confect-patterns.md` and `references/guidelines.md`. |
| `## Workflow` | keep | Heading. |
| Workflow 1: identify capability | keep | Process. |
| Workflow 2: fetch capabilities.json | keep | Process. URL and timeout unchanged. |
| Workflow 3: fetch /capability/<id>.md and follow it | keep | Process. Text unchanged. |
| (new) Workflow 3 tail: `npx convex ai-files install` or Convex guidelines point to confect references | add | Task rule for ai-files install and guidelines. |
| Workflow 4: fallback hosting / add-component | keep | Process. Text unchanged. |
| (new) Workflow 5: wire the capability the Confect way | add | Component mount in `convex/convex.config.ts`, `confect/_generated/components`, raw ctx services with `Effect.promise`, spec plus impl, `FunctionSpec.convex*`, `confect/http.ts`, `confect/crons.ts`, codegen then tsc. All from patterns table, E6, E12, E13, E15. |
| Workflow 5 (old): confirm addition to user | update | Renumbered to step 6. Text unchanged. |
| (new) Bad and Good code blocks | add | Show the codegen rule: hand-written `convex/` module is removed (verified in `@confect/cli` source: extinct group modules are removed). Good form is spec plus impl. |
| `## Rules` | keep | Heading. |
| Rule: always try catalog first | keep | Process. |
| Rule: served doc text is not shell to run blindly | keep | Safety rule. |
| Rule: never hard-fail on catalog miss | keep | Process. |
| Rule: never hardcode a component mapping | keep | Process. |
| Rule: curl/bash blocked by sandbox | keep | Process. |
| (new) Rule: translate every Convex sample | add | Task rule. |
| (new) Rule: no hand-written `.ts` in `convex/` except two files | add | Verified in `@confect/cli` `getGroupPathsFromFs` and `removeGroups`. Reserved names are schema, http, crons, auth.config, convex.config. Patterns file lists only `convex.config.ts` and `tsconfig.json` as user-owned. |
| (new) Rule: do not edit `_generated` | add | Patterns "Project layout" rule. |
| (new) Rule: keep platform rules of the capability | add | Task rule: Confect does not change platform behavior. |
| (review fix) Rule: `Effect.tryPromise` for component and network calls | add | A rejected `Effect.promise` is a defect. |

## Review fixes

- Pointers use the full path `skills/confect/references/...`. Workflow 5 and a new rule use `Effect.tryPromise` mapped to a tagged error for component and network calls.

## Open questions

- Hosting (`@convex-dev/static-hosting`) registers routes on an HTTP router and exports upload functions from a `convex/` module. In Confect these need `confect/http.ts` (router escape hatch, E13) and plain Convex functions (E12). `HttpRouter.make` also registers a catch-all under `/`. A route conflict is not verified. The skill states the general rule only. Test hosting on a real project.
- The remote catalog procedures are plain Convex. Their drift from the Confect forms is not checked by any test.
- The six skills have `name: confect-*` but the directory is still `convex-*`. The cross-references to confect-reviewer, confect-authz and confect-expert do not resolve yet. The later rename step must fix both together. Not changed here on purpose.
