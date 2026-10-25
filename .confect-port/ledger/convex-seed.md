# Ledger: skills/convex-seed/SKILL.md (code samples translated)

Counts: kept 8, updated 6, added 7, removed 0.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 2 name | update | convex-seed -> confect-seed. Directory name kept. |
| 3 description | update | Triggers on Confect work. Names internal mutation spec and impl. |
| 6 GENERATED comment | keep | Upstream line. |
| (new) adapted comment | add | Notes the adaptation. |
| 8 title | keep | Unchanged. |
| 10 intro | update | internalMutation -> internal mutation (FunctionSpec.internalMutation). convex import kept. |
| (new) read-first paragraph | add | Points to confect/references/confect-patterns.md only. The guidelines.md file and the confect skill do not exist. E2 and E4 kept. |
| 12 Workflow heading | keep | Unchanged. |
| 14 internalMutation fixtures, npx convex run | update | Spec plus impl with DatabaseWriter. Codegen first. convex run kept. Concrete form: module path with slashes (billing/invoices:run), example seed:run. |
| 15 bulk import | update | npx convex import kept. Added: import writes raw values, use Encoded form (Schema transformations). |
| 16 idempotent | keep | Platform rule. Code sample shows bounded clear-then-insert with a scheduled continuation. |
| 17 verify row counts | keep | Platform only. |
| (new) spec code sample | add | internalMutation spec. Marked // Good:. Verified FunctionSpec.internalMutation in patterns. |
| (new) impl code sample | add | Bounded take, delete, scheduled continuation when the batch is full, upsert of the parent user with GetByIndexFailure, insert with required authorId (matches E1 notes table), Effect.orDie. delete and insert signatures checked in @confect/server dist/DatabaseWriter.d.ts. |
| 19 Rules heading | keep | Unchanged. |
| 21 seed via internalMutation | update | internal mutation spec, table schemas. |
| 22 idempotent | keep | Platform only. |
| 23 no secrets/PII | keep | Platform only. |
| (new) transaction and batch rule | add | One seed run is one transaction. Batch and schedule with Scheduler.runAfter. Convex limits kept. |
| (new) upsert rule | add | get(indexName, ...) fails with GetByIndexFailure. Recover with catchTag. From patterns concept map. |
| (new) Bad examples | add | Two // Bad: samples: a public mutation as a seed, and an unbounded collect() clear. Repo style rule. |
