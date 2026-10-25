---
name: confect-deploy-guard
description: "Classify + announce the target Convex deployment before any deployment-affecting command in a Confect project (convex deploy, convex dev, convex run, convex env set); fresh explicit consent for prod actions; session read-only mode."
---

<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Deployment target guard

Platform only. Confect does not change deployments. The deploy, env, and MCP rules are the same as in plain Convex. The Convex guidelines in the project's `convex/_generated/ai/guidelines.md` when that file exists (`npx convex ai-files install` creates it) stay the source for platform rules.

Deployments are not interchangeable, and most incidents start with a command aimed at the wrong one. Every Convex project has several (personal dev, preview, prod — often across multiple projects on one machine). This guard is the standing discipline: identify, announce, then act — and treat prod as consent-gated, per action, per session.

## Workflow

1. IDENTIFY before you act: read `CONVEX_DEPLOYMENT` in .env.local, `convex.json`, and whether `CONVEX_DEPLOY_KEY` is set; or call the official Convex MCP `status` tool. Classify the target: local-anonymous | dev | preview | prod. If two sources disagree, resolve before proceeding.
2. ANNOUNCE in one line before any deployment-affecting command: `target: dev (joyful-capybara-123, personal dev)`. Never run the command in the same breath as discovering the target — announce first.
3. PROD needs a FRESH explicit yes: before `npx convex deploy` (when it resolves to prod), `npx convex run --prod`, `env set` on prod, snapshot `import`/`export` on prod, or starting the MCP with prod access — state exactly what will change on which deployment and get an explicit yes in THIS session. A yes given earlier, or for a different target, does not carry.
4. MCP safety defaults: start the official MCP scoped non-prod (`--deployment dev`). The two prod flags are DIFFERENT risk levels — keep them split: a read-only prod audit (advisor/insights reading data/logs/insights) passes ONLY `--cautiously-allow-production-pii` (read tools); `--dangerously-enable-production-deployments` (which enables MUTATING prod tools) stays OFF unless the user explicitly asked to CHANGE prod this session. Never pair them by default — 'look at prod' must not silently grant 'mutate prod'.
5. READ-ONLY session mode: when the user says 'read-only' / 'don't change anything', honor it absolutely for the rest of the session — no deploy, no env set/remove, no mutations via `run`, no imports; start the MCP with `--disable-tools run,envSet,envRemove`.
6. Wrong-deployment diagnosis: when a deploy 'didn't change anything', do NOT re-deploy harder. Re-run step 1 — the deploy almost certainly landed on a different deployment than the one being observed. In a Confect project, check the target first. If the target is correct, check that codegen is fresh (step 9).
7. Ambiguity = stop: if you cannot determine which deployment a command will hit, find out (status tool) — never guess.
8. In a Confect project, `npx confect codegen` and `npx confect dev` write files only. They do not deploy. `npx convex dev` and `npx convex deploy` push the generated `convex/` modules. Run steps 1 and 2 before the Convex commands.
9. A deploy needs fresh codegen. Run `npx confect codegen`, then `tsc --noEmit`, then deploy. A stale `convex/` directory deploys old functions to the correct target, and the deploy can look like it did nothing. Check this only after the target is confirmed (see step 6).

## Rules

- Classify and announce the target BEFORE every deployment-affecting command — identification and action are two separate steps.
- Prod consent is per-action, per-target, per-session: state what changes where, get a fresh explicit yes.
- Keep the two prod MCP flags split by risk: --cautiously-allow-production-pii (read-only) for an audit; --dangerously-enable-production-deployments (mutating) only when the user explicitly asks to change prod. Both are user-spoken-only; default every MCP start to a non-prod deployment selector.
- Read-only mode, once requested, is absolute for the session — including 'harmless' mutations.
- A deploy that seemed to do nothing means the WRONG deployment changed — diagnose the target, don't re-run. In a Confect project, when the target is correct, the cause can be stale codegen. Run codegen, then deploy once.
- This guard composes: ship, env, migrate, and seed run it as their step 0; it is not itself a deploy tool.
- Do not edit the generated files in `convex/` to change a deploy. Change the Confect source in `confect/`, then run codegen.
