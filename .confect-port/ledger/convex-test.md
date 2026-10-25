# Ledger: skills/convex-test/SKILL.md (platform: code samples translated)

Counts: kept 5, updated 12, added 10, removed 0.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1-4 frontmatter name | update | convex-test -> confect-test. Directory name kept. |
| 3 description | update | Triggers on Confect work (confect/ directory, specs, impls). |
| 6 GENERATED comment | keep | Upstream line. Kept for small syncs. |
| (new) adapted comment | add | Notes the Confect adaptation. |
| 8 title | update | "Generate Confect tests". |
| 10 intro | update | convex-test + vitest -> @confect/test + @effect/vitest. Kept the list of test targets. Added typed errors and HTTP routes. |
| (new) read-first paragraph | add | Points to references/confect-patterns.md (E16) and to the project convex/_generated/ai/guidelines.md. The fork has no references/guidelines.md. Convex guidelines stay the platform source. |
| 12 Workflow heading | keep | Unchanged. |
| 14 install | update | Add @confect/test, vitest and @effect/vitest (4.x, peer vitest >=5 <6). convex-test stays. |
| (new) test helper step | add | Helper file with TestConfect_.TestConfect<typeof confectSchema>() and TestConfect_.layer(...). Glob matches ts and js, excludes d.ts (codegen writes js plus d.ts). Also fixed the same glob in patterns E16. |
| 15 convexTest(schema), t.run, t.query/t.mutation | update | TestConfect service, confect.run, confect.query/mutation/action with refs. |
| 16 withIdentity | update | confect.withIdentity. Kept. |
| 16 error paths | update | Effect.flip and Effect.catchTag. |
| 16 t.finishInProgressScheduledFunctions | update | Same name on the TestConfect service. Verified in @confect/test dist/TestConfect.d.ts. Added finishAllScheduledFunctions (takes advanceTimers). |
| (new) HTTP route coverage | add | confect.fetch (replaces t.fetch). |
| 17 run vitest, deterministic | keep | Platform-neutral. |
| (new) code sample | add | Neutral notes domain, from patterns E16. Marked // Bad: and // Good:. |
| (new) Bad example | add | String function name and live deployment. Repo rule for examples. |
| (new) Setup notes (6 bullets) | add | From E16 test notes: inline convex-test, exclude *.spec.ts, vite/client reference, types entry (with reason), components (no @confect/test support; escape hatch is raw convex-test registerComponent), codegen first. |
| 19 Rules heading | keep | Unchanged. |
| 21 Use convex-test (in-memory) | update | @confect/test (in-memory). Not a live deployment: kept. |
| 22 Cover auth + error paths | keep | Unchanged. |
| 23 Keep tests deterministic | update | Same rule. Added vi.useFakeTimers() and the advanceTimers detail. |
| (new) use refs rule | add | Confect refs replace string names. |
| (new) typed error assertion rule | add | Defect vs typed error. Function and middleware error schemas both decode as typed errors. From E2. |
