# Ledger: convex-advisor (SKILL.md, 32 lines)

Source file: skills/convex-advisor/SKILL.md. Directory name kept; a later step renames it.
Code content: API mentions in root-cause fixes (`.collect()`, `.filter()`, `.withIndex`, `.take`, `.paginate`). No code samples upstream.

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter `name: convex-advisor` | update | `confect-advisor`. |
| Frontmatter `description` | update | Confect app with a `confect/` directory. Insights, root cause, findings kept. |
| GENERATED comment | keep | Unchanged upstream line. |
| (new) "Adapted to Confect" comment | add | Same marker as the adapted skills. |
| `# Live-deployment advisor` | keep | Unchanged. |
| Intro paragraph (insights tool, event kinds, findings bus) | keep | Platform and MCP facts. Unchanged. |
| (new) Paragraph: deployment runs generated `convex/`; code is in `confect/`; pointers | add | Explains the source mapping. Points to patterns and guidelines. |
| Workflow 1: GUARD deploy-guard step 0-1 | keep | Platform and process. Unchanged. |
| Workflow 2: GATHER via MCP (status, insights, tables, functionSpec) | update | Only change: "offering convex-reviewer" to "offering confect-reviewer". All MCP and availability facts kept. |
| (new) Workflow 3: MAP event to Confect source | add | Function id `group:name` maps to `<group>.spec.ts` and `<group>.impl.ts`. `functionSpec` shows the generated surface. Middleware reads count. From patterns "Scope and sources" and E2, E5. |
| Workflow 3 (old): ROOT-CAUSE intro | update | Renumbered to step 4. Text unchanged. |
| Root cause bullet: read thresholds and limits | update | Raw `.collect()`, unindexed `.filter()`, `.withIndex` became Confect forms: no `.filter()` on queries (Gaps), `.stream()` plus `Stream.filter`, `paginate` filter, `.index("by_creation_time")` scan, `.index(name, range)`, `.take(n)`, `publicPaginatedQuery`. "convex-expert patterns" became "confect-expert patterns". Aggregate fix kept. |
| Root cause bullet: OCC | update | Fix list kept (sharded-counter, narrow read set, workpool). Added that a `get` or index read puts the document in the read set. |
| Root cause bullet: repeated failures in logs | update | Classes kept. Added Confect failure shape: error not in the `error` schema becomes a defect (E2). |
| Workflow 4: EMIT findings | keep | Findings bus rule. Renumbered to step 5. Text unchanged. |
| Workflow 5: REPORT | update | Renumbered to step 6. Added: cite `confect/*.impl.ts` line; run codegen and tsc after a fix. Rest unchanged. |
| Workflow 6: scope discipline | update | Renumbered to step 7. `convex-authz`, `convex-reviewer` became `confect-authz`, `confect-reviewer`. `sentinel` kept (platform skill without prefix). |
| (new) Bad and Good code blocks | add | Show scan then filter versus index range plus `take`. From E3 and Gaps (`.filter()`). |
| Rule: evidence-not-vibes | update | "offer convex-reviewer" became "offer confect-reviewer". |
| Rule: read-only by construction | keep | Platform and safety. Unchanged. |
| Rule: root-cause in code before reporting | keep | Unchanged. |
| Rule: emit on findings bus | keep | Unchanged. |
| Rule: severity from event kind | keep | Unchanged. |
| Rule: stay in lane | update | Skill names to `confect-authz`, `confect-reviewer`. |
| Rule: prefer component fixes | update | Kept. Added: call a component client with the raw ctx service in the same mutation (E6). |
| (new) Rule: report Confect source, never fix generated file | add | Patterns "Project layout" rule. |
| (new) Rule: `Stream.filter` and `paginate` filter do not cut rows read | add | Patterns Gaps row for `.filter()`. |

## Review fixes

- Pointers use the full path. The read-threshold root-cause bullet is now a short list.

## Open questions

- `deploy-guard` and `sentinel` are bare names upstream. They were not renamed. The later directory rename step must decide if they become `confect-*`.
- The `finding.schema.json` bus lives outside this fork. Not changed.
- The six skills have `name: confect-*` but the directory is still `convex-*`. The cross-references to confect-reviewer, confect-authz and confect-expert do not resolve yet. The later rename step must fix both together. Not changed here on purpose.
