# Ledger: convex-reviewer (SKILL.md, 26 lines; no references/ directory)

Source file: skills/convex-reviewer/SKILL.md. Directory name kept; a later step renames it.
This skill has code content (API mentions only, no code samples). It is not "platform only".

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter `name: convex-reviewer` | update | Confect form: `confect-reviewer`. |
| Frontmatter `description` | update | Triggers on `confect/` directory and Confect specs, impls, tables, middleware. Keeps security, auth, performance, pattern scope. "validators" became "schemas". |
| GENERATED comment | update | File is hand-edited. Comment reworded to say adapted by hand. |
| `# Convex Code Reviewer` | update | `# Confect Code Reviewer`. |
| Intro paragraph (structured review, severity levels) | update | Same text with Confect nouns: schemas, table design. Severity scale kept. |
| (new) Paragraph: Confect does not change platform behavior; pointers | add | Task rule: point to confect skill `references/confect-patterns.md` and `references/guidelines.md`. Convex guidelines stay the platform source. |
| Workflow 1: public functions check `ctx.auth.getUserIdentity()` | update | `Auth` service `auth.getUserIdentity`, or middleware that provides the user (patterns table, E5). |
| Workflow 1: verify ownership before reads/writes | keep | Platform rule, unchanged. |
| Workflow 1: no client-provided user IDs trusted | keep | Platform rule, unchanged. |
| Workflow 1: scheduled functions target `internal.*` not `api.*` | update | `refs.internal.*` and `refs.public.*` (E6, E7). |
| Workflow 2: no `.filter()`, `withIndex` required | update | Every query names `.index(...)`. `.stream()` plus `Stream.filter` and the `paginate` filter argument do not cut rows read. Raw ctx `.filter()` is checked in pass 5. |
| Workflow 2: foreign-key fields have indexes | keep | Platform rule. |
| Workflow 2: no `Date.now()` in query handlers | update | Query impls. Added `Clock.currentTimeMillis` (reads real time per Version notes). |
| Workflow 2: `.collect()` not on unbounded queries | update | Added Confect bounded forms: `take(n)`, `publicPaginatedQuery`. |
| Workflow 3: args and returns validators on every public function | update | `args` and `returns` schemas on every public function spec. Paginated specs take `item`. Plain `convex*` specs use Convex validators. |
| Workflow 3: no `any` types | update | TypeScript `any` kept, now explicit (`as any`, `any` parameter). Added `Schema.Any`, `Schema.Unknown` (patterns table: avoid). |
| (new) Workflow 3: `Schema.Number` | add | Task rule and patterns Validators table: use `Schema.Finite`. |
| Workflow 3: promises are awaited | update | Effects are yielded. An Effect not yielded never runs. |
| Workflow 3: arrays bounded (<8192) | keep | Platform limit, unchanged. |
| (new) Workflow 5: escape hatch pass | add | Upstream raw checks for plain `convex*` specs and raw ctx (E12, Gaps): validators, auth, index, `api.*` scheduling, awaited promises. |
| (new) Workflow 4: Confect structure pass | add | Checks Confect-only failure modes: spec imports of `@confect/server`, `error` schemas, `GetByIdFailure`/`GetByIndexFailure` mapping, impl layer completeness, edits in `_generated`. All from patterns E2, E3, E5 and Project layout rules. |
| Workflow 4: report grouped by severity | update | Renumbered to step 6. Text unchanged. |
| Rule: missing auth checks Critical | keep | Text kept. Added one sentence: group without auth middleware needs an `Auth` check per function. |
| (new) Rule: client-provided user ID Critical | add | Platform auth rule in patterns "Platform rules that still apply": use `tokenIdentifier`. |
| Rule: `.filter()` Important, full table scan | update | Reworded for Confect: reads without an index range. |
| Rule: `Date.now()` in query handlers Important, breaks reactivity | update | Confect names. Added the Version-notes fact that 10.0.0 does not stub `Date.now()`. |
| Rule: missing args or returns validators Important | update | Schemas. Added "omitted args means no arguments". |
| Rule: scheduling to `api.*` Important | update | `refs.public.*` / `refs.internal.*`. |
| (new) Rules: missing `error` schema, `Schema.Number`, `Effect.fail(new ...)`, unbounded collect/array | add | Confect idioms from the task rules (Effect v4 preferences) and patterns E2. Unbounded collect/array restates platform limits. |
| Rule: always explain why | keep | Unchanged. |
| (new) Example finding (Bad/Good) | add | One neutral-domain sample of the index-range rule, in two code blocks. Bad comment says it reads up to 1000 rows. Uses `reader.table(..).index(..).take(..)` from patterns E3. |

## Counts
- kept: 6
- updated: 18
- added: 7
- removed: 0

## Open questions
- Pointer to `references/guidelines.md` replaced by the "Platform rules that still apply" section of `confect-patterns.md`. Convex guidelines are not copied into the fork.
- Frontmatter name is `confect-reviewer`, directory is `convex-reviewer`. All other fork skills use the same split. The planned directory rename step must handle all skills together.
- The Bad example uses the `by_creation_time` index name from the patterns table ("Table scan"). Confirm it with a compile check.
