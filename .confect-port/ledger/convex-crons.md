# Ledger: convex-crons (platform plus code)

Source: skills/convex-crons/SKILL.md (23 lines). Sources of truth: confect-patterns.md E15, CronJobs and CronJob exports.

| Source line or block | Action | Reason |
| --- | --- | --- |
| 2 name: convex-crons | update | Renamed to confect-crons. Directory name kept. |
| 3 description | update | Triggers on Confect work (confect/crons.ts, CronJobs, refs.internal). |
| 6 GENERATED comment | keep | Unchanged upstream line. Open question: file is now hand edited. |
| 8 title | keep | Still correct. |
| 10 intro: convex/crons.ts | update | Path is confect/crons.ts. |
| (new) pointer paragraph | add | Points to confect-patterns.md E15 and references/guidelines.md. |
| 14 Create convex/crons.ts with cronJobs() | update | CronJobs.make() default export with .add(CronJob.make(...)). |
| 15 Schedule internal functions, never api.* | update | refs.internal.* and refs.public.*. Spec uses FunctionSpec.internal*. |
| 16 idempotent, small | keep | Platform rule. |
| 17 Verify in dashboard | keep | Platform step. Renumbered to 5. |
| (new) codegen plus tsc step | add | Codegen writes convex/. Patterns workflow rule 4. |
| (new) code sample | add | Confect form of crons.ts (Cron and Duration targets). |
| 21 internal.* never api.* | update | Confect refs form. |
| 22 small + idempotent | keep | Platform rule. |
| 23 no tight polling | keep | Platform rule. |
| (new) Cron minute granularity, UTC, Duration whole units | add | Verified in patterns E15. |
| (new) do not use convex/crons.ts or cronJobs() | add | Codegen owns convex/. |
| (review fix) guidelines pointer | update | Pointed to a missing references/guidelines.md. Now points to the project's convex/_generated/ai/guidelines.md when present. |
| 6 GENERATED comment | update | Review fix: banner was false. Replaced with the Adapted to Confect comment. |
| (review fix) code sample | update | Added Bad example (raw cronJobs, time zone). |
| (review fix) codegen owns convex/ rule | update | Codegen skips convex.config.ts and tsconfig.json. User owns both. |
| (review fix) time zone rule | add | Cron with a non-UTC zone throws at module load (CronJobs.ts). |
