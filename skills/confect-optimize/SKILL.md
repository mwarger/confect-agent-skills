---
name: confect-optimize
description: "Audit and optimize an existing Confect app (Effect on Convex, a confect/ directory): security, scale, upgrades, observability."
---

<!-- GENERATED from convex-agents content/capabilities/optimize.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Audit and optimize an existing Confect app

The remediation WORKFLOW for an existing app: open with a scored assessment, then act on it — upgrade stale components and set up observability — plan-then-confirm-then-apply. The assessment itself is delegated to confect-launch-readiness (the findings-bus scorer); optimize's distinct value is the actions it takes on the result.

Before you change code, read `confect/references/confect-patterns.md`. The Convex guidelines stay the source for platform rules.

## Workflow

1. Detect the app: a `confect/` directory (specs, impls, `confect/tables/`), the generated `convex/` directory, and whether it's an anonymous or cloud deployment.
2. ASSESS via `confect-launch-readiness` — one scored, deduped report across authz/reviewer/advisor/insights with an ordered fix plan. Do not re-run those passes by hand; optimize consumes confect-launch-readiness's report rather than re-implementing the audit.
3. UPGRADE: run `check-updates` against the pinned `@convex-dev/*` components and fold stale-component (staleness-class) findings into the same plan. Also check the `@confect/*` packages and `effect`. Every `@confect/*` package in the project must use the same major version. This includes `core`, `server`, `cli` and `test`. It also includes the client packages `@confect/js` and `@confect/foldkit` when the project uses them. Both client packages need `@confect/core` at the same major version. `@confect/server` needs `convex` ^1.45.0. `@confect/test` needs `convex-test` >=0.0.50 <0.1.0. Check these peer ranges before you upgrade. Read the package sources for API changes before you upgrade. Run `npx confect codegen` and `tsc --noEmit` after each upgrade.
4. OBSERVABILITY: if the readiness report flagged an observability gap (no prod error capture), offer to install `confect-sentinel`.
5. Present the combined prioritized plan — the confect-launch-readiness score + the fix plan + upgrades + observability, security/data-loss first — and apply only on explicit confirmation, dispatching each fix to its fixCapability.
6. After applying, re-run the confect-launch-readiness assessment and show the score delta.

## Rules

- Read-only first. Present a plan and CONFIRM before changing any file.
- Delegate the audit to confect-launch-readiness (the findings-bus scorer); don't re-implement reviewer/advisor/insights inline — optimize's job is acting on the report (upgrades + observability), not re-scoring.
- Prioritize security and data-loss risks above style, following confect-launch-readiness's ordering.
- Never auto-land changes on someone's existing prod app; re-assess after applying and show the score moved.
- Change files in `confect/` and `convex/convex.config.ts`. Do not hand-edit `confect/_generated/` or the generated modules in `convex/`.
