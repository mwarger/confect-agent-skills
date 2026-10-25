# Ledger: convex-launch-readiness -> confect-launch-readiness (SKILL.md)

Skill type: platform only for scoring; small Confect additions (pass names, locus mapping, codegen + tsc gate). No code samples. No references/ directory.
Sources: confect-patterns.md (project layout, checklist last item).

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1, 4 frontmatter delimiters | keep | structure |
| 2 name | update | now confect-launch-readiness (directory name kept) |
| 3 description | update | Confect audits; confect/ trigger |
| 6 GENERATED comment | keep | unchanged upstream line |
| 8 title | keep | |
| 10 intro | keep | bus and scoring are not Convex API |
| (new) after 10 | add | note (no "Platform only" lead in the skill text); confect references; codegen + tsc --noEmit gate |
| 12 Workflow heading | keep | |
| 14 step 1 | update | adds confect/ dir and auth foundation forms |
| 15 step 2 lead | keep | |
| 16 authz pass | update | convex-authz becomes confect-authz |
| 17 reviewer pass | update | confect-reviewer; Schema validators; typed error vs defect |
| 18 advisor pass | update | confect-advisor |
| 19 insights pass | update | confect-insights |
| 20 concurrency line | keep | |
| 21 step 3 | update | adds code locus (FunctionSpec + FunctionImpl) mapped to the <group>:<name> key; dedup rules kept |
| 22 step 4 | keep | scoring formula unchanged |
| 23 step 5 | keep | |
| 24 step 6 | update | fixer names renamed; migrate-rehearse becomes confect-migrate-rehearse; re-run codegen and tsc |
| 25 step 7 | keep | |
| 27 Rules heading | keep | |
| 29 rule 1 | keep | |
| 30 rule 2 | keep | |
| 31 rule 3 | update | code locus is the Confect spec and impl |
| 32 rule 4 | keep | |
| 33 rule 5 | keep | |
| 34 rule 6 | keep | |
| 35 rule 7 | keep | |

## Review fixes

- Review fix: the path `confect/references/guidelines.md` does not exist in the fork (only `confect-patterns.md` does). The text now points to `confect-patterns.md` only and says the platform rules are the same as in plain Convex. About 20 other skill files still name the missing path. Fix them in a later step.

- Known risk: frontmatter `name` is `confect-launch-readiness` but the directory is `skills/convex-launch-readiness/`. The Agent Skills format expects `name` to equal the directory name. A strict loader can reject or rename the skill. Cross-references by name point to a different directory name. Fix in one step later: rename the directories, or keep this risk recorded.
