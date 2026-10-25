# Ledger: convex-cost (SKILL.md, 29 lines)

Source file: skills/convex-cost/SKILL.md. Directory name kept; a later step renames it.
Code content: API mentions in fixes (`.collect()`, `.withIndex`, `.paginate`, `.take`). No code samples upstream.

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter `name: convex-cost` | update | `confect-cost`. |
| Frontmatter `description` | update | Confect app with a `confect/` directory. Ranking, growth curve, confirm-cost kept. |
| GENERATED comment | keep | Unchanged upstream line. |
| (new) "Adapted to Confect" comment | add | Same marker as the adapted skills. |
| `# Preview what this app will cost` | keep | Unchanged. |
| Intro paragraph | update | Only change: `convex-advisor` to `confect-advisor`. Money lens and confirm-cost text kept. |
| (new) Paragraph: Confect does not change billing; map function ids to `confect/`; pointers | add | Task rule for pointers. Explains the source mapping. |
| Workflow 1: GUARD, read-only, insights cloud and user auth only | keep | Unchanged. |
| Workflow 2: GATHER via MCP (insights, tables, functionSpec); no-traffic estimate | update | All platform facts kept. Added the id to `confect/<group>.impl.ts` mapping. `.collect()` shape also names a wide `by_creation_time` scan (patterns: every query names an index). |
| Workflow 3: ATTRIBUTE bytes per call times volume | keep | Unchanged. |
| Workflow 4: PROJECT growth curve | keep | Unchanged. |
| Workflow 5: cheapest fix per driver | update | `index + .withIndex` became an index plus an index range in `.index(name, range)`. `.paginate` through `publicPaginatedQuery`. `convex-expert/convex-advisor` became `confect-expert/confect-advisor`. Added: no `.filter()` on queries; `Stream.filter` does not cut rows read (Gaps). |
| Workflow 6: CONFIRM-COST for paid actions | keep | Unchanged. |
| Workflow 7: REPORT, relative guidance, pricing page | keep | Unchanged. |
| (new) Bad and Good code blocks | add | `collect()` versus `index` range plus `take(50)`. From E3. |
| Rule: cost = data read times volume | keep | Unchanged. |
| Rule: read insights; price shapes when no traffic | keep | Unchanged. |
| Rule: growth curve, not dollars | keep | Unchanged. |
| Rule: every driver names cheapest fix and emits finding | update | `convex-expert/advisor` became `confect-expert/advisor`. |
| Rule: confirm-cost for paid actions | keep | Unchanged. |
| Rule: read-only over dev/prod; composes advisor | update | `convex-advisor` became `confect-advisor`. |
| (new) Rule: clock read in a query evicts it from cache | add | Patterns Version notes: `Date.now()` is not stubbed in 10.0.0. Platform cost fact: the query loses caching. |
| (new) Rule: report Confect source, never fix generated file | add | Patterns "Project layout" rule. |
| (new) Rule: many action `runQuery` or `runMutation` calls add cost | add | Patterns E6: each call is a separate transaction. |

## Review fixes

- Pointers use the full path. The Bad comment now matches the code (one author, not the whole table). Steps 2 and 5 are short sentences and lists.

## Open questions

- The cache-eviction cost claim comes from the patterns Version notes. It was not measured.
- `deploy-guard` is a bare name upstream and was not renamed.
- The six skills have `name: confect-*` but the directory is still `convex-*`. The cross-references to confect-reviewer, confect-authz and confect-expert do not resolve yet. The later rename step must fix both together. Not changed here on purpose.
