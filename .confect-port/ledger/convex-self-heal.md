# Ledger: skills/convex-self-heal/SKILL.md (platform mostly: code references translated)

Counts: kept 13, updated 16, added 3, removed 0.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 2 name | update | convex-self-heal -> confect-self-heal. Directory name kept. |
| 3 description | update | Triggers on Confect work. Adds codegen to the certify list. |
| 6 GENERATED comment | keep | Upstream line. |
| (new) adapted comment | add | Notes the adaptation. |
| 8 title | keep | Unchanged. |
| 10 intro | keep | Platform narrative. Skill names inside are capability names. |
| (new) read-first paragraph | add | Points to confect/references/confect-patterns.md only. The guidelines.md file and the confect skill do not exist. |
| 12 Workflow heading | keep | Unchanged. |
| 14 step 1 GUARD | update | deploy-guard -> confect-deploy-guard. |
| 15 step 2 CAPTURE | update | sentinel -> confect-sentinel. |
| 16 step 3 TRIAGE | update | hand to confect-env. Added: generated function name maps to Confect group path, find spec and impl in confect/. |
| 17 step 4 ROOT-CAUSE | update | insights, advisor, reviewer, authz renamed to confect-*. |
| 18 step 5 REPAIR | update | Fixer names renamed. Added: change confect/, not generated files. |
| 19 step 6 CERTIFY lead | keep | Unchanged. |
| 20 (a) tsc --noEmit | update | Added npx confect codegen first. Codegen alone is not a type check. |
| 21 (b) schema or data, migrate-rehearse | update | Names confect/tables/ and confect-migrate-rehearse. Gate kept. |
| 22 (c) convex-test case | update | convex-test -> @confect/test TestConfect. Effect.flip with _tag for typed failure, Effect.exit for defect. E16. |
| 23 (d) no-regression | keep | Platform only. |
| 24 could not certify | keep | Platform only. |
| 25 step 7 PROPOSE | update | Evidence list adds codegen. |
| 26 step 8 PROMOTE | update | confect-deploy-guard. insights vs logs rule kept. |
| 27 step 9 BOUND | update | Kept. Added Confect forms: Schema change, .index(...), codegen owns convex/schema.ts, index backfill blocks deploy. Notes that staged: true is a Convex option that Confect does not support (Table.index has no options argument), so a Confect index fix always blocks the deploy and the promote step must plan for it. |
| 29 Rules heading | keep | Unchanged. |
| 31 human keeps merge | keep | Platform only. |
| 32 certify before proposing | update | Adds codegen. Names renamed. |
| 33 triage first | update | env -> confect-env. |
| 34 repair on branch | update | deploy-guard -> confect-deploy-guard. |
| 35 pre-approved classes | keep | Platform only. |
| 36 close the loop | keep | Platform only. |
| 37 append-only record | keep | Platform only. |
| 38 root cause unclear | keep | Platform only. |
| (new) no hiding rule | add | Do not use Effect.orDie to turn a typed error into a defect for a green test. |
