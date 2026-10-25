# Ledger: convex-verify -> confect-verify (SKILL.md)

Skill type: has code content (test tooling). No references/ directory exists.
Sources: confect-patterns.md (E16, E5, concept map row for t.withIdentity), @confect/test 10.0.0 TestConfect.d.ts.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1, 4 (frontmatter delimiters) | keep | structure |
| 2 name: convex-verify | update | now confect-verify (directory name kept) |
| 3 description | update | triggers on Confect work; names the withIdentity method of the TestConfect service; keeps seed/multi-user/negative authz meaning |
| 6 GENERATED comment | keep | unchanged upstream line, keeps syncs small (see open questions) |
| 8 title | keep | no Convex API |
| 10 intro paragraph | update | convex-test wrapped by @confect/test; rest kept |
| (new) after 10 | add | points to confect skill confect-patterns.md (guidelines.md does not exist, so it is not named); Convex guidelines stay the platform source |
| 12 Workflow heading | keep | |
| 14 step 1 | update | add: find function via FunctionSpec and refs |
| 15 step 2 | update | convex-test setup becomes @confect/test layer; add @confect/test, @effect/vitest, spec exclude, codegen first, TestConfect.layer; keep edge-runtime, inline, @edge-runtime/vm facts; the unverified import.meta.glob failure claim is dropped (modules are passed explicitly); local test/TestConfect.ts module, service tag and layer explained; test capability -> confect-test |
| 16 step 3 | update | t.run ctx.db.insert becomes confect.run + DatabaseWriter; "validators" becomes Schema decoding |
| 17 step 4 | update | t.withIdentity becomes confect.withIdentity; add issuer, query/mutation/action call form; keep three identities |
| 18-19 step 5 and positive bullet | update | api.x.y becomes refs.public.x.y; calls return Effect |
| 20 negative bullet | update | rejects.toThrow regex replaced by typed error: Effect.flip + _tag, catchTag; keep rule that wrong caller must be refused |
| (new) defect bullet | add | defects (Effect.orDie) are not seen by Effect.flip; use Effect.exit; prefer typed spec error |
| (new) HTTP and plain Convex bullets | add | HTTP refusals use confect.fetch and res.status; FunctionSpec.convex* refusals are defects (Effect.exit) |
| 21 data-scope bullet | keep | platform-neutral |
| 22 step 6 | update | add tsc --noEmit gate (patterns: codegen alone not enough); bus finding text kept |
| 23 step 7 | update | convex-authz -> confect-authz |
| 25 Rules heading | keep | |
| 27 rule 1 | keep | platform-neutral |
| 28 rule 2 | update | t.withIdentity -> confect.withIdentity |
| 29 rule 3 | keep | |
| 30 rule 4 | update | "convex-test to run" -> "test layer to run"; add vite/client reference rule |
| (new) typed error rule | add | Confect refusals are typed errors, not message text |
| 31 rule 5 | update | in-process with @confect/test; confect-test capability |
| 32 rule 6 | update | convex-authz/convex-expert -> confect-authz/confect-expert |
| 33 rule 7 | keep | |
| (new) component rule | add | from patterns E16 test notes |
| 34 rule 8 | update | test capability -> confect-test |

Counts: kept 9, updated 16, added 5 (incl. blocks), removed 0.

Open question: frontmatter name is confect-verify but the directory is convex-verify. The Agent Skills spec says they should match. This applies to the whole fork and needs one decision (rename all directories or keep).
