# Ledger: convex-migrate -> confect-migrate (SKILL.md)

Skill type: has Confect content (schema in table files; backfill as a plain Convex function). No code samples (prose only; see open questions). No references/ directory.
Sources: confect-patterns.md (E1, E12, component rows, validators table, Gaps: staged indexes).

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1, 4 frontmatter delimiters | keep | structure |
| 2 name | update | now confect-migrate (directory name kept) |
| 3 description | update | Confect schema in confect/tables/ |
| 6 GENERATED comment | keep | unchanged upstream line |
| 8 title | keep | |
| 10 intro | keep | platform method |
| (new) after 10 | add | points to confect references; same platform gate; Confect changes where schema and backfill live |
| (new) Where the code goes | add | tables dir, optionalKey, convex.config.ts, components registry, plain-function backfill and spec constructor, raw ctx.db, codegen + tsc order |
| 12 Workflow heading | keep | |
| 14 step 1 | update | optional becomes Schema.optionalKey in table file; codegen and tsc; deploy |
| 15 step 2 | keep | |
| 16 step 3 | keep | |
| 17 step 4 | update | table file change; codegen and tsc; update impls |
| 19 Rules heading | keep | |
| 21 rule 1 | keep | |
| 22 rule 2 | keep | |
| 23 rule 3 | keep | |
| (new) after 23 | add | do not edit convex/schema.ts or _generated |
| (new) after 23 | add | index backfill block rule; no staged indexes in Confect 10.0.0 |
| (new) after 23 | add | backfill is a plain Convex function, not a whole-table FunctionImpl |

## Review fixes

- Review fix: the path `confect/references/guidelines.md` does not exist in the fork (only `confect-patterns.md` does). The text now points to `confect-patterns.md` only and says the platform rules are the same as in plain Convex. About 20 other skill files still name the missing path. Fix them in a later step.

- Known risk: frontmatter `name` is `confect-migrate` but the directory is `skills/convex-migrate/`. The Agent Skills format expects `name` to equal the directory name. A strict loader can reject or rename the skill. Cross-references by name point to a different directory name. Fix in one step later: rename the directories, or keep this risk recorded.
- Review fix: step 1 now runs `tsc --noEmit` after codegen, as step 4 does.
