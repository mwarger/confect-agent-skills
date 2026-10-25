# Agent Instructions

Rules for contributing to this repository.

## Skill Philosophy

Skills in this repository should be narrowly focused on a specific task or
workflow.

A good skill helps an agent take a concrete action, for example:

- set up authentication
- design a schema
- create a Convex function
- plan a migration

Do not create skills that mainly provide generic background information or broad
documentation. If content is mostly reference material, it should usually live
in docs, not as a standalone skill.

Reference material is fine inside a skill when it directly helps complete the
task the skill is for.

Before contributing, review the core Agent Skills docs:

- [Overview](https://agentskills.io/home)
- [What are skills?](https://agentskills.io/what-are-skills)
- [Specification](https://agentskills.io/specification)
- [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- [Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)

If the skill uses bundled scripts, also review
[Using scripts in skills](https://agentskills.io/skill-creation/using-scripts).

## Style Guidelines

- **No emojis** in any markdown files or code comments
- Use `Yes/No` in tables instead of checkmarks or emoji
- Keep examples concise and focused
- Use `// Bad:` and `// Good:` comments in code examples
- Follow existing file patterns in `skills/`

## Confect rules

This fork adapts the Convex skills to Confect 10.0.0 and Effect v4.

- Use `skills/confect/references/confect-patterns.md` for each Convex to
  Confect mapping. Do not invent a new pattern when the file has one.
- Verify every Confect export in the installed `@confect/*` package source
  (`src/` and `dist/*.d.ts`). Do not guess an API name. When the docs and the
  source differ, the source wins.
- Keep every Convex platform fact: limits, index rules, transactions, the
  scheduler, crons, HTTP actions, file storage, auth, components, the CLI, and
  deploy rules. Confect changes how code is written. It does not change how
  Convex behaves.
- When Confect does not wrap a Convex feature, say so. Show the supported
  escape hatch.
- Never copy code, names, tables, business rules, or comments from a private
  app. Use neutral example domains: notes, messages, tasks, users.
- Write new or changed text in short, plain sentences. Use one instruction per
  sentence.
- Leave unchanged upstream lines exactly as they are. This keeps upstream syncs
  small.
- Record each change to an upstream file in `.confect-port/ledger/`.

## File Structure

Each skill should have:

- Clear heading with brief description
- "When to Use" section
- Code examples showing bad vs good patterns
- Complete, runnable examples
- Checklist at the end

## Maintenance

- If skills are added, removed, renamed, or substantially repositioned, update
  the root `README.md`
- Keep the skill list in the root `README.md` in sync with the current contents
  of `skills/`

## Validation

- Do not assume a skill is good just because it reads well
- Validate skills by trying to use them in a realistic temp project or sandbox
- Push the task as far as possible without human help
- If human intervention is required, stop and ask explicitly for the exact step
  needed
- After the run, feed what you learned back into the skill with clearer
  instructions, steps, gotchas, or validation checks
- For UI-facing skills, validate the browser flow as part of skill authoring,
  not just the code or CLI steps

## Code Examples

```ts
// Bad: description of the problem

// Good: description of the solution
```
