# Ledger: skills/convex-monitor/SKILL.md (platform only)

Counts: kept 10, updated 3, added 5, removed 0.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 2 name | update | convex-monitor -> confect-monitor. Directory name kept. |
| 3 description | update | Triggers on Confect work (confect/ directory). |
| 6 GENERATED comment | keep | Upstream line. Kept for small syncs. |
| (new) adapted comment | add | Notes the adaptation and "platform only". |
| 8 title | keep | Platform only. |
| 10 intro | keep | Platform only. Events come from the Convex deployment. |
| (new) read-first paragraph | add | Points to confect/references/confect-patterns.md only. The guidelines.md file and the confect skill do not exist. Convex guidelines stay the platform source. |
| 12 Workflow heading | keep | Unchanged. |
| 14 wait_for_event step | keep | MCP tool. No code. |
| 15 event kinds | update | "see sentinel" -> "see confect-sentinel". Rest kept. |
| (new) fix-in-confect step | add | Fix source in confect/, not generated files. Codegen after a change to a spec, a table file, a group file, crons.ts, http.ts or auth.ts. Then tsc. |
| 16 poll loop step | keep | Platform only. Renumbered 3 -> 4. |
| 18 Rules heading | keep | Unchanged. |
| 20 prefer blocking tool | keep | Platform only. |
| 21 fixed event schema | keep | Platform only. |
| 22 prod events need Sentinel | keep | Platform only. |
| (new) generated names rule | add | Event function names follow Confect group paths. |
| (new) typed error vs defect rule | add | Expected failure vs defect (Effect). |
