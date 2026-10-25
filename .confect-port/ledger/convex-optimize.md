# Ledger: skills/convex-optimize/SKILL.md (platform only: no code samples)

Counts: kept 5, updated 12, added 3, removed 0.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 2 name | update | convex-optimize -> confect-optimize. Directory name kept. |
| 3 description | update | Triggers on Confect work. |
| 6 GENERATED comment | keep | Upstream line. |
| (new) adapted comment | add | Notes the adaptation. |
| 8 title | update | "Confect app". |
| 10 intro | update | launch-readiness -> confect-launch-readiness. Rest kept. |
| (new) read-first paragraph | add | Points to confect/references/confect-patterns.md only. The guidelines.md file and the confect skill do not exist. |
| 12 Workflow heading | keep | Unchanged. |
| 14 detect app | update | Adds confect/ directory, specs, impls, tables, generated convex/. Anonymous vs cloud kept. |
| 15 assess via launch-readiness | update | Cross-reference rename only. |
| 16 upgrade @convex-dev components | update | Kept. Added @confect/* and effect check, one major version for every @confect/* package, including @confect/js and @confect/foldkit (peers on @confect/core ^10). Peer ranges: convex ^1.45.0 for server, convex-test >=0.0.50 <0.1.0 for test. Codegen and tsc after upgrade. |
| 17 observability, sentinel | update | Cross-reference rename. |
| 18 combined plan | update | Cross-reference rename. Apply-on-confirm kept. |
| 19 re-run assessment | update | Cross-reference rename. |
| 21 Rules heading | keep | Unchanged. |
| 23 read-only first | keep | Platform only. |
| 24 delegate audit | update | Cross-reference rename. |
| 25 prioritize security | update | Cross-reference rename. |
| 26 never auto-land | keep | Platform only. |
| (new) edit scope rule | add | Edit confect/ and convex.config.ts. Never hand-edit generated files. |
