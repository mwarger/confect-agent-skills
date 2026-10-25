# Ledger: convex-explain-app -> confect-explain-app (SKILL.md)

Skill type: has Confect content (reads tables and specs). No code samples. No references/ directory.
Sources: confect-patterns.md (project layout, concept map, E1, E2, E5, E13, E15).

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1, 4 frontmatter delimiters | keep | structure |
| 2 name | update | now confect-explain-app (directory name kept) |
| 3 description | update | triggers on a confect/ project; source is table files and specs |
| 6 GENERATED comment | keep | unchanged upstream line |
| 8 title | update | "Convex app" becomes "Confect app" |
| 10 intro | update | schema becomes table files; functionSpec/exports become group specs |
| (new) after 10 | add | points to confect skill guidelines.md and confect-patterns.md; generated files are not the source |
| 12 Workflow heading | keep | |
| 14 step 1 | update | add confect/ layout; only convex.config.ts and tsconfig.json are hand-written; function names follow group paths; MCP functionSpec kept |
| 15 step 2 | update | schema.ts becomes confect/tables/*.ts; v.id becomes Id(); index builders named; GenericId for _storage added; FK-graph example kept |
| 16 step 3 | update | query/mutation/action become FunctionSpec public/internal constructors; impls read; plain convex* functions and error schemas added; public/internal rationale kept |
| 17 step 4 | update | auth.config.ts becomes confect/auth.ts; requireOwner becomes middleware or Auth service; middleware gaps (plain functions, subgroups) added; do-not-audit rule kept (confect-authz) |
| 18 step 5 | update | confect/http.ts, confect/crons.ts, Config for env vars; component list kept; catch-all HTTP action note added |
| 19 step 6 | update | example flow uses Confect names (middleware, DatabaseWriter, index name) |
| 20 step 7 | update | next verbs renamed to confect-*; "never invent" rule kept |
| 22 Rules heading | keep | |
| 24 rule 1 | update | source of truth is tables, specs, impls, functionSpec |
| (new) after 24 | add | do not treat generated files as authored code |
| 25 rule 2 | keep | platform-neutral |
| 26 rule 3 | update | convex-authz becomes confect-authz |
| 27 rule 4 | keep | platform-neutral |
| 28 rule 5 | keep | platform-neutral |
| 29 rule 6 | keep | platform-neutral |

## Review fixes

- Review fix: the path `confect/references/guidelines.md` does not exist in the fork (only `confect-patterns.md` does). The text now points to `confect-patterns.md` only and says the platform rules are the same as in plain Convex. About 20 other skill files still name the missing path. Fix them in a later step.

- Known risk: frontmatter `name` is `confect-explain-app` but the directory is `skills/convex-explain-app/`. The Agent Skills format expects `name` to equal the directory name. A strict loader can reject or rename the skill. Cross-references by name point to a different directory name. Fix in one step later: rename the directories, or keep this risk recorded.
- Review fix: step 4 now names function-level middleware (`FunctionSpec.publicQuery({...}).middleware(Spec)`) next to group-level middleware.
- Review fix: step 5 now says Confect registers one catch-all HTTP action under `/` and that the `HttpApi` groups hold the real paths.
