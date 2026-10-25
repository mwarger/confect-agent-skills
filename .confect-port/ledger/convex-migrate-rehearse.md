# Ledger: convex-migrate-rehearse -> confect-migrate-rehearse (SKILL.md)

Skill type: platform only for the flow; Confect changes: schema location and codegen before deploy. No code samples. No references/ directory.
Sources: confect-patterns.md (project layout workflow, E12, Gaps).

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1, 4 frontmatter delimiters | keep | structure |
| 2 name | update | now confect-migrate-rehearse (directory name kept) |
| 3 description | update | Confect schema in confect/tables/ |
| 6 GENERATED comment | keep | unchanged upstream line |
| 8 title | keep | |
| 10 intro | update | composes confect-deploy-guard and confect-migrate; platform text kept |
| (new) after 10 | add | confect references; gate and commands are platform; codegen + tsc before every deploy |
| 12 Workflow heading | keep | |
| 14 step 0 | keep | preview key facts are platform |
| 15 step 1 | keep | deploy-guard text kept (see open questions) |
| 16 step 2 | keep | export command is platform |
| 17 step 3 | update | table files instead of schema.ts; codegen and tsc before deploy; import facts kept |
| 18 step 4 | update | codegen + tsc before each push; optionalKey; backfill as plain Convex function; all deploy flag facts kept |
| 19 step 5 | update | Confect function names; typed error vs defect |
| 20 step 6 | keep | promote and rollback facts are platform |
| 21 step 7 | keep | |
| 23 Rules heading | keep | |
| 25 rule 1 | update | table files instead of schema.ts |
| (new) after 25 | add | codegen and tsc before every deploy |
| 26 rule 2 | keep | |
| 27 rule 3 | keep | |
| 28 rule 4 | keep | |
| 29 rule 5 | keep | |
| 30 rule 6 | keep | |
| 31 rule 7 | update | migrate becomes confect-migrate |

## Review fixes

- Review fix: the path `confect/references/guidelines.md` does not exist in the fork (only `confect-patterns.md` does). The text now points to `confect-patterns.md` only and says the platform rules are the same as in plain Convex. About 20 other skill files still name the missing path. Fix them in a later step.

- Known risk: frontmatter `name` is `confect-migrate-rehearse` but the directory is `skills/convex-migrate-rehearse/`. The Agent Skills format expects `name` to equal the directory name. A strict loader can reject or rename the skill. Cross-references by name point to a different directory name. Fix in one step later: rename the directories, or keep this risk recorded.
- Review fix: step 3 now runs `npx confect codegen` and `tsc --noEmit` before `--preview-create`, so every deploy step names both commands.
