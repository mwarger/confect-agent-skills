---
name: confect-self-heal
description: "Production error → triaged, root-caused, repaired, and certified (tsc + codegen + rehearsal + reproduce-then-gone) fix PR for a human to merge in a Confect app (a confect/ directory) — then confirm the error stops recurring. Never auto-merges."
---

<!-- GENERATED from convex-agents content/capabilities/self-heal.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Gated production self-healing loop

Sentry/Datadog/Vercel can go error→investigate→draft-PR, but they treat the backend as opaque and stop at the human merge gate with an unverified diff. Convex can do the step they can't: because the error rows live in the user's own deployment and the fix can be rehearsed on a preview of that deployment, the platform certifies the fix against real invariants before anyone reviews it. This capability is the composition capstone — it wires sentinel (capture) → the findings bus (diagnose) → the fixers (repair) → migrate-rehearse/tsc/probe (certify) → a human PR (decide) → deploy-guard (promote). The human keeps the merge button; the machine does everything up to and including proving the fix works.

Before you repair code, read `confect/references/confect-patterns.md`. The Convex guidelines stay the source for platform rules.

## Workflow

1. GUARD: confect-deploy-guard — this loop reads prod and PROPOSES prod changes; classify + announce the deployment and get the standing consent for the loop's scope up front (what classes of fix it may auto-prepare vs must always defer). Never auto-merge; the human merge is the fixed boundary.
2. CAPTURE: require confect-sentinel (prod errors in the user's own deployment, redacted at write time). If absent, offer to install it and stop — there is nothing to heal without capture.
3. TRIAGE a new/ recurring error: pull it via the official MCP (data/run-once-query over the sentinel table, or the monitor's prod_error event). Classify: transient (retry/ignore — do NOT open a PR for a one-off network blip), config (env/secret — hand to confect-env, never guess a secret), or a code/schema defect (proceed). The function name in the error is the generated Convex name. It follows the Confect group path. Find the spec and the impl in `confect/` from that path.
4. ROOT-CAUSE on the findings bus: run the relevant audit pass on the implicated function — confect-insights (the failing requests + stacks), confect-advisor (if it's a read-limit/OCC cause), confect-reviewer/confect-authz (if it's a logic/authz defect). Produce a bus finding with evidence (the stack + the reproducing input) and a fixCapability. If root cause is unclear, STOP and report — a wrong fix is worse than an open error.
5. REPAIR via the finding's fixCapability (confect-authz, reviewer fixers, confect-expert for perf) on a branch — never on prod directly. Change files in `confect/`. Do not hand-edit `confect/_generated/` or the generated modules in `convex/`.
6. CERTIFY against the backend's own invariants BEFORE proposing (this is the differentiator — do not skip any that apply):
   (a) `npx confect codegen` succeeds, then `tsc --noEmit` clean (codegen success alone is not a type check);
   (b) if the fix touches schema/data (a file in `confect/tables/`, or a data backfill), run it through confect-migrate-rehearse on a preview seeded with a prod snapshot — the schema-conformance gate must pass on real-shaped data;
   (c) reproduce-then-confirm-gone: replay the error's triggering input against the fixed code (a `@confect/test` case with `TestConfect`, or an MCP run on the preview) and assert the failure no longer occurs. Assert a typed failure with `Effect.flip` and its `_tag`. Assert a defect with `Effect.exit`. See E16 in `confect-patterns.md`;
   (d) no-regression: the finding must be gone AND no new bus finding introduced on the touched function.
   A fix that fails any applicable certification is NOT proposed — it's reported as 'attempted, could not certify' with what failed.
7. PROPOSE, never merge: open a PR (or a diff for review) containing the fix, the certification evidence (codegen and tsc result, rehearsal outcome, the reproduced-then-gone assertion), the original error + finding, and the reversibility note. Label the change class. The human reviews and merges.
8. PROMOTE on merge via confect-deploy-guard's prod consent; after deploy, re-check the sentinel table + `logs` (failures) to confirm that error signature stops recurring (do NOT use `insights` for this — it tracks only OCC/read-limit perf events, not arbitrary error signatures) — the loop is only closed when the error stops recurring in prod. If it recurs, reopen with the new evidence.
9. BOUND it: only classes the user pre-approved in step 1 are auto-prepared (default-safe set: validator fixes, missing-index adds, ownership-check adds, non-destructive backfills); anything destructive, security-sensitive beyond an added check, or ambiguous is always deferred to explicit human direction. Log every action to an append-only record so the loop is auditable. In Confect, a validator fix is a `Schema` change in a table file or a spec. A missing-index add is a `.index(...)` call on the table builder. Codegen owns `convex/schema.ts`. A new index on a large table blocks the deploy until backfill completes. Convex can backfill without blocking when the index is declared with `staged: true`. Confect does not support this. `Table.index(name, fields)` has no options argument. A Confect fix that adds an index always blocks the deploy. Plan the promote step for that.

## Rules

- The human keeps the merge button — this loop prepares and certifies fixes, it NEVER auto-merges or auto-deploys to prod (matches the industry boundary: no credible system ships unattended prod auto-merge).
- Certify before proposing: codegen + tsc + (schema→confect-migrate-rehearse on a prod-snapshot preview) + reproduce-then-confirm-the-failure-is-gone + no new bus finding. An uncertified fix is reported as 'could not certify', never proposed as done.
- Triage first: transient blips get retried/ignored, config errors go to confect-env (never guess a secret), only real code/schema defects enter the repair loop.
- Repair on a branch/preview, never on prod directly; promote only through confect-deploy-guard's fresh prod consent.
- Only pre-approved fix classes are auto-prepared (default-safe: validator/index/ownership/non-destructive backfill); destructive or ambiguous changes are always deferred to the human.
- Close the loop for real: after merge+deploy, confirm the error signature stops recurring via the sentinel table + logs (not insights, which only sees perf events); reopen if it persists.
- Every action is logged to an append-only, auditable record; data residency stays in the user's own deployment (sentinel discipline).
- If root cause is unclear, STOP and report — an uncertain fix is worse than an open, visible error.
- Do not hide a failure to certify it. Do not turn a typed error into a defect with `Effect.orDie` to make a test pass.
