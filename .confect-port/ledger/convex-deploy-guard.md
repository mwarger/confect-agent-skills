# Ledger: convex-deploy-guard (platform only)

Source: skills/convex-deploy-guard/SKILL.md (29 lines). No Convex code samples. All steps are CLI and MCP rules.

| Source line or block | Action | Reason |
| --- | --- | --- |
| 2 name | update | Renamed to confect-deploy-guard. Directory name kept. |
| 3 description | update | Mentions Confect commands. Platform meaning kept. |
| 6 GENERATED comment | keep | Unchanged upstream line. |
| 8 title | keep | Still correct. |
| (new) platform only note | add | Skill is platform only. Points to references/guidelines.md. |
| 10 intro | keep | Platform text. |
| 14 step 1 IDENTIFY | keep | Platform. |
| 15 step 2 ANNOUNCE | keep | Platform. |
| 16 step 3 PROD fresh yes | keep | Platform. |
| 17 step 4 MCP safety | keep | Platform. |
| 18 step 5 read-only mode | keep | Platform. |
| 19 step 6 wrong deployment | keep | Platform. |
| 20 step 7 ambiguity | keep | Platform. |
| (new) step 8 confect codegen and dev write files only | add | Confect command semantics. Verified in patterns Workflow. |
| (new) step 9 fresh codegen before deploy | add | Stale convex/ deploys old functions. |
| 24-29 all six rules | keep | Platform. |
| (new) do not edit generated convex/ files | add | Patterns Project layout rule. |
| (review fix) guidelines pointer | update | Pointed to a missing references/guidelines.md. Now points to the project's convex/_generated/ai/guidelines.md when present. |
| 6 GENERATED comment | update | Review fix: banner was false. Replaced with the Adapted to Confect comment. |
| 3 description | update | Review fix: lists convex commands only. Codegen and dev do not deploy. |
| 19 step 6 and step 9 and rule | update | Review fix: diagnosis order is target first, then codegen freshness. Rule has a Confect note. |
