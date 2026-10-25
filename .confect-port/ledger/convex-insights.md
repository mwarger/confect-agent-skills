# Ledger: convex-insights -> confect-insights (SKILL.md)

Skill type: platform only for the MCP tools; small Confect additions (names, typed errors vs defects, logging). No code samples. No references/ directory.
Sources: confect-patterns.md (project layout note on function names, E2 typed errors, concept map row for console.log).

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1, 4 frontmatter delimiters | keep | structure |
| 2 name | update | now confect-insights (directory name kept) |
| 3 description | update | Confect app; adds typed errors vs defects |
| 6 GENERATED comment | keep | unchanged upstream line |
| 8 title | keep | no Convex API |
| 10 intro | keep | MCP tools (logs, insights, functionSpec, status) are platform tools |
| (new) after 10 | add | note (no "Platform only" lead in the skill text); names follow group paths; Effect.log replaces console.log; points to confect references |
| 12 Workflow heading | keep | |
| 14 step 1 | keep | deploy-guard text kept (skill reference unchanged, see open questions) |
| 15 step 2 | update | adds function name form <group>:<name>; all tool limits kept |
| 16 step 3 lead | keep | |
| 17 failures view | update | adds typed error vs defect split (ConvexError data, _tag); rest kept |
| 18 health view | update | convex-advisor becomes confect-advisor |
| 19 trace view | keep | |
| 20 step 4 | keep | |
| 21 step 5 | keep | |
| 22 step 6 | update | adds codegen deploy flow and renamed group file changes function name |
| 23 step 7 | update | advisor, reviewer, authz renamed to confect-*; bus text kept |
| 25 Rules heading | keep | |
| 27 rule 1 | keep | |
| 28 rule 2 | keep | |
| 29 rule 3 | keep | |
| 30 rule 4 | keep | |
| (new) after 30 | add | count typed errors and defects separately |
| 17 failures view (added text) | add | two edge cases: no `error` schema turns every failure into a defect (`Effect.orDie`); a thrown `ConvexError` defect reaches the client bare (for example `InvalidCursor`) |
| 31 rule 5 | keep | |
| 32 rule 6 | update | convex-advisor becomes confect-advisor |

## Review fixes

- Review fix: the path `confect/references/guidelines.md` does not exist in the fork (only `confect-patterns.md` does). The text now points to `confect-patterns.md` only and says the platform rules are the same as in plain Convex. About 20 other skill files still name the missing path. Fix them in a later step.

- Known risk: frontmatter `name` is `confect-insights` but the directory is `skills/convex-insights/`. The Agent Skills format expects `name` to equal the directory name. A strict loader can reject or rename the skill. Cross-references by name point to a different directory name. Fix in one step later: rename the directories, or keep this risk recorded.
