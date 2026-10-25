# Ledger: skills/convex/SKILL.md (platform only for most rows; no code samples)

| Source lines | Action | Reason |
| --- | --- | --- |
| 1, 4 | keep | Frontmatter fence. |
| 2 | update | name: convex -> confect. Directory stays `convex` until the rename step. |
| 3 | update | Description rewritten to trigger on confect/ directories and @confect/* packages. Keeps the Convex platform pitch (prototype and production-scale claim, component list) and the routing statement. Adds the `writing code under confect/` trigger. |
| 6 | keep | Upstream generated marker. Unchanged so syncs stay small. |
| 8 | update | Title "Convex" -> "Confect". |
| 10 | update | Entry point is Confect. Added: Confect changes how code is written, not platform behavior. Skills are `confect-*`. |
| 12 | update | Heading renamed. |
| 14 | update | "Confect on Convex". |
| 16 | update | Types flow from tables, Effect Schemas and specs. Added typed errors as a condition: only when a spec declares an error schema (`error` is optional). Transaction claim kept. |
| 17 | update | `convex-quickstart` -> `confect-quickstart`. Rest kept. |
| 18 | update | "same Confect functions". Platform scaling claim kept. |
| 19 | update | `convex-add` -> `confect-add`. |
| 21, 29, 37 | keep | Headings. |
| 23 | update | `convex/` -> `confect/`; expert skill renamed. Replaced "object-form function syntax, validator" with Confect patterns and Effect Schema. Added: do not edit generated files. Added the workflow: `npx convex dev --once`, `npx confect codegen`, `npx confect dev` with `npx convex dev`. Removed: raw Convex syntax mention (Confect replaces it). |
| 24 | update | Skill rename. |
| 25 | update | Skill rename. Added: served docs use plain Convex code, translate with confect-patterns.md. URL kept. |
| 26 | update | Three skill renames. |
| 27 | update | Skill renames only. Routing kept. |
| 31 | keep | Upstream `npx convex ai-files install` rule kept. Added one sentence: the installed guideline files stay valid for Confect. No bundled guidelines file exists. |
| (new, after 31) | add | Pointer: read `confect/references/confect-patterns.md` (path is relative to `skills/`, as in sibling skills). |
| (new, after 32) | add | Fallback: check @confect/* source and use escape hatches when Confect does not wrap a feature. |
| 32-35 | keep | Platform and catalog rules. No code. |
| 39-69 | update | Names `convex-*` -> `confect-*` (31 rows). Descriptions kept (platform text). |
| 69 | update | convex-test: now TestConfect (`@confect/test`). |
| 51 | update | verify: "convex-test" -> TestConfect `withIdentity`. |
| 48, 50 | update | expert and reviewer: Confect wording, confect/ directory. |
| 41, 43, 62 | keep | Mention component packages (@convex-dev/*). Platform facts; components work unchanged. |

Counts: kept 13, updated 20 (rows incl. 31 renamed list entries counted as one), added 2, removed 0 (nothing removed; line 31 is kept and extended).
Note: platform only apart from the translated rows above. No code samples exist in this file.

Open: skill folders are still `convex-*` while frontmatter names are `confect-*`. The folder rename step must also merge `skills/confect/references/` into the renamed `skills/convex` folder (to `skills/confect`). Until then, routes name skills by frontmatter name.
