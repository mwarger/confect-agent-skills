# Ledger: convex-env (platform plus code)

Source: skills/convex-env/SKILL.md (24 lines). Sources of truth: patterns E10, Config.Redacted verified in effect 4.0.0 Config.d.ts.

| Source line or block | Action | Reason |
| --- | --- | --- |
| 2 name | update | Renamed to confect-env. Directory name kept. |
| 3 description | update | Config, Redacted. |
| 6 GENERATED comment | keep | Unchanged upstream line. |
| 8 title | keep | Unchanged. |
| 10 intro process.env | update | Effect Config. |
| (new) pointer paragraph | add | E10, provider, always-set vars, Node actions use default provider. |
| 14 convex env set | keep | Same CLI. |
| 15 process.env in actions | update | Config.String and Config.Redacted. Actions-only rule kept as upstream. |
| 16 never commit | keep | Platform. |
| 17 env list --names-only | keep | Platform. |
| (new) Bad/Good sample | add | process.env vs Config. |
| 21 secrets in env vars | keep | Platform. |
| 22 process.env only in actions | update | Config only in actions. Do not use process.env. |
| 23 own values per deployment | keep | Platform. |
| 24 names-only | keep | Platform. |
| (new) Config.Array/Record, typed env gap, missing var failure | add | Patterns E10 and Gaps. |
| (review fix) guidelines pointer | update | Pointed to a missing references/guidelines.md. Now points to the project's convex/_generated/ai/guidelines.md when present. |
| 6 GENERATED comment | update | Review fix: banner was false. Replaced with the Adapted to Confect comment. |
