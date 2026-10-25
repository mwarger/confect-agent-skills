# Ledger: convex-backup (SKILL.md, 33 lines)

Source file: skills/convex-backup/SKILL.md. Directory name kept; a later step renames it.
Classification: platform only. No Convex code samples. Only CLI and MCP commands. Confect adds two facts: codegen before deploy, and Effect decode on read.

Verified: `@confect/cli` 10.0.0 has two commands, `codegen` and `dev`. It has no deploy command.

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter `name: convex-backup` | update | `confect-backup`. |
| Frontmatter `description` | update | Names the Confect app. States "Platform only". Drill, RPO, runbook kept. |
| GENERATED comment | keep | Unchanged upstream line. |
| (new) "Adapted to Confect" comment | add | Same marker as the adapted skills. |
| `# Back up — and prove the restore works` | keep | Unchanged. |
| Intro paragraph (two halves, drill, migrate-rehearse primitives) | keep | Platform text. Unchanged. |
| (new) Paragraph: no Confect code; two Confect changes; pointer to guidelines | add | Task rule: say "platform only" and point to the guidelines. |
| Workflow 1: GUARD deploy-guard | keep | Unchanged. |
| Workflow 2: `npx convex export --path`, `--include-file-storage` | keep | CLI text. Unchanged. |
| Workflow 3: schedule matched to RPO, daily export, retention | keep | Unchanged. |
| Workflow 4(a): Preview Deploy Key precondition | keep | Unchanged. |
| Workflow 4(b): create preview from current code | update | Added `npx confect codegen` first, so `convex/` matches `confect/`. Added: Confect has no deploy command. Deploy command unchanged. |
| Workflow 4(c): `npx convex import ... --deployment ... --replace` | keep | Unchanged, including the `--preview-name` note. |
| Workflow 4(d): assert recovery, row counts, spot-checks | update | All text kept. Added: MCP and `runOneoffQuery` show the encoded form; call one Confect read; `DocumentDecodeError` counts as a failed drill (patterns E3, Validators table). |
| Workflow 5: REPORT, prod runbook, `--prod`, deploy-guard | keep | Unchanged. |
| Workflow 6: HYGIENE | keep | Unchanged. |
| (new) Bad and Good shell blocks (codegen before deploy) | add | Show the order rule. |
| Rule: backup never restored is a hope | keep | Unchanged. |
| Rule: drill restores to throwaway preview | keep | Unchanged. |
| Rule: assert recovery, 0 rows is failed | keep | Unchanged. |
| Rule: failed drill is most valuable | keep | Unchanged. |
| Rule: schedule matched to RPO | keep | Unchanged. |
| Rule: snapshots are sensitive | keep | Unchanged. |
| Rule: shares migrate-rehearse mechanics | keep | Unchanged. |
| (new) Rule: codegen before every `npx convex deploy` | add | Deploy reads generated `convex/` modules. |
| (new) Rule: import checks Convex validators only; test a Confect read after import | add | Patterns Validators table: Effect checks such as `isMaxLength` run in Confect, not in the Convex validator. The import-time check is an inference from that fact, not tested. |

## Review fixes

- Pointer uses the full path.

## Open questions

- The claim "import checks only the Convex validators" is inferred from the patterns table. It was not tested with `npx convex import`.
- Whether `convex/` is committed or generated in CI differs by project. The skill says "run codegen first" in both cases.
- The six skills have `name: confect-*` but the directory is still `convex-*`. The cross-references to confect-reviewer, confect-authz and confect-expert do not resolve yet. The later rename step must fix both together. Not changed here on purpose.
