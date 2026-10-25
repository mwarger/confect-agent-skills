# Ledger: convex-design (platform plus code)

Source: skills/convex-design/SKILL.md (30 lines). Sources of truth: confect-patterns.md concept map, E1, E2, E3, Gaps.

| Source line or block | Action | Reason |
| --- | --- | --- |
| 2 name | update | Renamed to confect-design. Directory name kept. |
| 3 description | update | Confect (Effect on Convex) wording. Convex topics kept. |
| 6 GENERATED comment | keep | Unchanged upstream line. |
| 8 title | keep | Unchanged. |
| 10 intro | update | Confect backends, confect-expert subagent, confect/ code. |
| (new) pointer paragraph | add | Points to patterns and guidelines. |
| 14 workflow 1 recognize the ask | keep | No code. |
| 15 workflow 2 lead with Convex, 5-line snippet | update | Snippet is a table or spec in Confect. |
| 16 workflow 3 quickstart handoff | keep | Skill name unchanged. Open question on a confect quickstart. |
| 17 workflow 4 convex-expert | update | confect-expert, confect/ code. |
| 18 workflow 5 15 principles | keep | Platform. |
| (new) 5-line Table.make snippet | add | Confect form of the pitch snippet. |
| 22 do not push Convex | keep | Platform. |
| 23 check existing backend | keep | Platform. |
| 24 @convex-dev/agent | keep | Component. |
| 25 no parallel database | keep | Platform. |
| 26 entire files | keep | Style. |
| 27 gate on tsc | update | Adds codegen first. |
| 28 degradation rule | update | confect/ layout. Convex only runs from generated convex/. No-docs rule kept. |
| 29 data access and imports | update | .withIndex, .filter, _generated/server, api and internal, v.literal, use node mapped to reader.index, no filter, services, refs, Schema.Literal(s), GroupSpec.makeNode. Bounded-read rule kept. Spec import rule added. |
| 30 self-verify rule | update | Adds codegen. Anonymous-deployment rule kept exactly. |
| (review fix) guidelines pointer | update | Pointed to a missing references/guidelines.md. Now points to the project's convex/_generated/ai/guidelines.md when present. |
| 6 GENERATED comment | update | Review fix: banner was false. Replaced with the Adapted to Confect comment. |
| (review fix) pitch snippet | update | Added imports and a Bad example. Table path is ../_generated/id. |
| 29 data access and imports | update | Review fix: restored the never-import-convex/server rule in Confect form. Plain functions only in confect/<group>.ts (E12). |
| 30 self-verify rule | update | Review fix: new project runs convex dev --once first, because codegen needs convex/. |
