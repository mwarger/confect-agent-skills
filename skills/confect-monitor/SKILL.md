---
name: confect-monitor
description: "Watch for the next dev/prod error or request in a Confect app (Effect on Convex, a confect/ directory) and react to it."
---

<!-- GENERATED from convex-agents content/capabilities/monitor.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform only: the events come from the Convex deployment. -->

# Watch for the next thing to react to

Block on the next typed event instead of polling. Races local error logs, deployment subscriptions, and Sentinel prod-error rows; returns the first to fire (or a quiet heartbeat).

Before you fix code, read `confect/references/confect-patterns.md`. The Convex guidelines stay the source for platform rules.

## Workflow

1. Call `wait_for_event` with {project_dir, event_kinds, timeout_ms}.
2. On kind=convex_error/next_error: decode and fix it. On kind=prod_error: triage (see confect-sentinel) and fix. On kind=feature_request: build it. On kind=quiet: loop.
3. Fix the source in `confect/`. Do not edit the generated `convex/` modules or `confect/_generated/`. Run `npx confect codegen` after you change a spec, a table file in `confect/tables/`, a group file (add or rename), `crons.ts`, `http.ts` or `auth.ts`. Run `tsc --noEmit` before you call the fix done.
4. Where a harness has no blocking MCP (e.g. Copilot cloud), the pack runs a poll loop with the SAME event contract — same behavior, different mechanism.

## Rules

- Prefer the blocking tool; fall back to a poll loop only where blocking MCP is weak.
- The event schema is fixed and versioned — the same trigger yields the same typed event.
- Prod events (kind=prod_error) require a deployed cloud app plus Sentinel.
- Function names in an event are the generated Convex names. They follow the Confect group paths. Find the spec and the impl in `confect/` from that path.
- A typed error declared in a spec is an expected failure. An unexpected failure (a defect) is the error to fix.
