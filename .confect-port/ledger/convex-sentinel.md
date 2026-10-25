# Ledger: skills/convex-sentinel/SKILL.md (platform only: component is plain Convex)

Counts: kept 10, updated 6, added 4, removed 0.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 2 name | update | convex-sentinel -> confect-sentinel. Directory name kept. |
| 3 description | update | Triggers on Confect work. |
| 6 GENERATED comment | keep | Upstream line. |
| (new) adapted comment | add | Notes the adaptation. |
| 8 title | keep | Unchanged. |
| 10 intro | keep | Platform only. |
| (new) read-first paragraph | add | Points to confect/references/confect-patterns.md only. The guidelines.md file and the confect skill do not exist. E6 kept. |
| 12 Workflow heading | keep | Unchanged. |
| 14 app.use(sentinel) | update | Kept. Added: you own convex.config.ts, codegen does not write it (patterns). |
| 15 client SDK | update | Kept. Added: client code, no Confect. |
| 16 redaction | keep | Platform only. |
| 17 read errors, monitor event | update | Kept. Added confect-monitor cross-reference. |
| 18 self-healing cron | update | Kept. Added confect-self-heal cross-reference. |
| 20 Rules heading | keep | Unchanged. |
| 22 redaction mandatory | keep | Platform only. |
| 23 data stays | keep | Platform only. |
| 24 sample and cap | keep | Platform only. |
| 25 prod needs cloud | keep | Platform only. |
| (new) component client rule | add | Confect does not wrap component clients. Raw ctx service plus Effect.promise. Gap table in patterns. |
| (new) typed error vs defect rule | add | Expected failure vs defect (Effect). |
