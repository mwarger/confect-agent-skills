# Ledger: convex-improve-convex-plugin -> confect-improve-confect-plugin (SKILL.md)

Skill type: platform only (a transcript upload helper). No code samples. No references/ directory.

| Source line(s) | Action | Reason |
| --- | --- | --- |
| 1, 4 frontmatter delimiters | keep | structure |
| 2 name | update | convex -> confect in both places gives confect-improve-confect-plugin (directory name kept) |
| 3 description | update | Confect session; "Convex team" kept (the anteater endpoint belongs to Convex); explicit-ask rule kept |
| 6 GENERATED comment | keep | unchanged upstream line |
| 8 title | update | name change |
| (new) after 8 | add | one line: no Convex or Confect code; the anteater endpoint belongs to Convex, which receives the transcript |
| 10 intro paragraph | keep | no Convex API; endpoint text unchanged |
| (new) after 10 | remove | scope claim removed: this fork does not control the Convex service, so it must not promise scope |
| 12 Workflow heading | keep | |
| 14 step 1 | keep | upstream helper command, unchanged |
| 15 step 2 | keep | consent flow, unchanged |
| 16 step 3 | keep | output markers, unchanged |
| 17 step 4 | keep | unchanged |
| 19 Rules heading | keep | |
| 21 rule 1 | keep | consent rule, unchanged |
| 22 rule 2 | keep | unchanged |
| 23 rule 3 | keep | unchanged |
| 24 rule 4 | keep | unchanged |

## Review fixes

- Known risk: frontmatter `name` is `confect-improve-confect-plugin` but the directory is `skills/convex-improve-convex-plugin/`. The Agent Skills format expects `name` to equal the directory name. A strict loader can reject or rename the skill. Cross-references by name point to a different directory name. Fix in one step later: rename the directories, or keep this risk recorded.
- Review fix: the process notes ("Only the names and one scope note changed") are removed from the skill text. The recipient is now stated as Convex.
