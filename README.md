# Confect Agent Skills

Agent skills for common Confect workflows.

## About this fork

This repository is a fork of
[get-convex/agent-skills](https://github.com/get-convex/agent-skills). It adapts
the Convex agent skills to [Confect](https://rjdellecese.gitbook.io/confect)
10.0.0 and Effect v4. Confect is Effect on Convex.

- Confect changes how you write code. It does not change how Convex behaves.
- The Convex platform rules come from upstream. These rules include limits,
  indexes, transactions, the scheduler, deploys, and the CLI. The fork keeps
  them.
- Each skill directory uses the `confect` form of the upstream name. For
  example, `convex-add` is `confect-add`.
- `skills/confect/references/confect-patterns.md` maps each Convex concept to
  its Confect equivalent.
- `.confect-port/ledger/` records the change to each upstream file. The ledger
  files use the upstream skill names.

To sync with upstream:

```bash
git fetch upstream
git merge upstream/main
```

Attribution: the upstream content is the work of the
[get-convex/agent-skills](https://github.com/get-convex/agent-skills) authors.
It is licensed under the Apache License 2.0. This fork keeps the upstream
[LICENSE](LICENSE). The fork changes are also licensed under the Apache License
2.0. Each changed file is a modified version of the upstream file.

## License

Unless otherwise noted, all original content in this repository is licensed
under the [Apache License 2.0](LICENSE). This includes the skill instructions,
reference documentation, and source code in revisions that include this license.
It does not retroactively change the licensing of earlier revisions.

Third-party materials remain under their respective licenses. See
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for details.

## Install

```bash
# Choose which skills you want
npx skills add mwarger/confect-agent-skills

# OR Install all skills
npx skills add mwarger/confect-agent-skills --all
```

## Usage

Skills are applied automatically when the agent determines they're relevant. How
you manually invoke them depends on your tool:

| Tool                     | Manual invocation |
| ------------------------ | ----------------- |
| Cursor                   | `/skill-name`     |
| VS Code (GitHub Copilot) | `/skill-name`     |
| Claude Code              | `/skill-name`     |
| Windsurf                 | `@skill-name`     |
| Codex (OpenAI)           | `$skill-name`     |

For example, to kick off auth setup in Cursor or Claude Code:

```
/confect-auth
```

In Windsurf:

```
@confect-auth
```

## Available Skills

- `confect` - Top-level entry point for Confect work. Recommends installing the
  Convex AI files, then routes to a more specific Confect skill.
- `confect-add` - Add a capability to the current Confect app.
- `confect-advisor` - Find the cause of read-limit and OCC events in code.
- `confect-agent` - Add an AI agent or RAG backend.
- `confect-auth` - Add authentication.
- `confect-authz` - Audit and harden authorization.
- `confect-backup` - Set up backups and run a restore drill.
- `confect-billing` - Add Stripe billing and payments.
- `confect-cost` - Preview spend and find the cheapest fix.
- `confect-create-component` - Build a reusable Convex component with clear
  boundaries.
- `confect-crons` - Add recurring scheduled jobs.
- `confect-deploy-guard` - Classify the target deployment before a deploy
  command.
- `confect-design` - Design a reactive, type-safe backend.
- `confect-docs` - Get the docs for the installed versions.
- `confect-domains` - Point a custom domain at the app.
- `confect-env` - Set deployment environment variables and secrets.
- `confect-expert` - Write and fix Confect backend code.
- `confect-explain-app` - Explain the data model, functions, and auth model of
  an app.
- `confect-improve-confect-plugin` - Send the session transcript to the Convex
  team for a post-mortem.
- `confect-insights` - Query the logs and health of a running app.
- `confect-launch-readiness` - Run all audits into one readiness report.
- `confect-migrate` - Migrate the schema and backfill data.
- `confect-migrate-rehearse` - Rehearse a migration on a preview deployment.
- `confect-monitor` - Watch for the next error or request.
- `confect-optimize` - Audit security, scale, upgrades, and observability.
- `confect-quickstart` - Start a new Confect app or add Confect to an existing
  frontend.
- `confect-reviewer` - Review Confect code before you ship it.
- `confect-seed` - Seed or import data.
- `confect-self-heal` - Find the cause of a production error and prepare a fix.
- `confect-sentinel` - Set up production error capture.
- `confect-suggest` - Suggest a component for a hand-built pattern.
- `confect-test` - Write tests with `@confect/test`.
- `confect-verify` - Prove a feature works, including the negative authz
  cases.

## Skill Philosophy

Skills in this repo should be laser-focused on a specific task or workflow.

A good skill helps an agent take action, for example:

- set up authentication
- create a component
- perform a migration
- diagnose performance issues

A skill should not exist just to provide generic background information. If
content is mostly reference material, it should usually live in documentation,
not as a standalone skill.

Reference material is still useful inside a skill, but only when it helps the
agent complete a concrete task.

## Contributing

Before contributing, review the core Agent Skills docs:

- [Overview](https://agentskills.io/home)
- [What are skills?](https://agentskills.io/what-are-skills)
- [Specification](https://agentskills.io/specification)
- [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- [Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)

If your skill bundles scripts, also read
[Using scripts in skills](https://agentskills.io/skill-creation/using-scripts).

Install dependencies once with `pnpm install`, then use:

- `pnpm format` to rewrite files
- `pnpm format-check` to verify formatting before opening a PR

### Validating Skills

Validate skills by trying to use them in a realistic temp project, not just by
reading them.

- Push the skill as far as possible with an agent in a throwaway directory
- If a human must intervene, ask explicitly for the exact action needed and then
  continue
- Record what worked, where the agent got stuck, and what confused the flow
- Feed those learnings back into the skill so the next run is better
- For UI-facing skills such as auth setup, validate the actual browser flow
  during skill development, not just code generation or a successful build

### Testing with Anthropic's Skill Creator

For a more rigorous approach, use Anthropic's
[skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)
skill. It provides a structured loop for testing and iterating on skills:

1. Draft or edit a skill
2. Define realistic test prompts and run the skill against them (with and
   without the skill as a baseline)
3. Grade the outputs with assertions and review them in a browser-based viewer
4. Improve the skill based on feedback, then repeat

It also includes a description optimizer that tunes the skill's frontmatter
description for better triggering accuracy across different phrasings.

Install it in Claude Code:

```bash
/install-skill https://github.com/anthropics/skills/tree/main/skills/skill-creator
```

This is especially useful when a skill is complex enough that reading it alone
won't tell you if it actually works well in practice.

Each skill follows the
[Agent Skills open standard](https://github.com/anthropics/skills):

1. Create a directory under `skills/` with the skill name
2. Add a `SKILL.md` file with YAML frontmatter:
   ```yaml
   ---
   name: skill-name
   description: Brief description
   ---
   ```
3. Include comprehensive examples with bad/good patterns
4. Add a checklist at the end of each skill
5. Update this root `README.md` whenever skills are added, removed, renamed, or
   substantially repositioned
