# Ledger: skills/convex-suggest/SKILL.md (platform mostly: detector hints translated)

Counts: kept 13, updated 5, added 4, removed 0.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 2 name | update | convex-suggest -> confect-suggest. Directory name kept. |
| 3 description | update | Triggers on Confect work. Rest kept. |
| 6 GENERATED comment | keep | Upstream line. |
| (new) adapted comment | add | Notes the adaptation. |
| 8 title | keep | Unchanged. |
| 10 intro | keep | Platform only. |
| (new) read-first paragraph | add | Points to confect/references/confect-patterns.md only. The guidelines.md file and the confect skill do not exist. E6 kept. |
| 12 Workflow heading | keep | Unchanged. |
| 14 observe passively | keep | Platform only. |
| 15 detector rules | update | All component mappings kept. Added Confect forms for .collect() count, Storage service, search(...). |
| 16 offer one suggestion | keep | Platform only. |
| 17 yes -> /add | update | /confect-add. Mount in convex.config.ts (you own it). Run codegen. |
| 18 no -> drop | keep | Platform only. |
| 20 Rules heading | keep | Unchanged. |
| 22 passive | keep | Platform only. |
| 23 one at a time | keep | Platform only. |
| 24 cite WHY | keep | Example about post.likes + 1 kept. It is a platform OCC fact, not a Convex API. |
| 25 never install | keep | Platform only. |
| 26 not already installed | keep | Platform only. |
| 27 generic questions | update | "Convex or Confect". |
| (new) component client rule | add | Raw ctx service, Effect.promise, same-mutation mirror rule. From patterns. |
| (new) static cron rule | add | CronJobs.make in confect/crons.ts for static crons. @convex-dev/crons for run-time crons. |
