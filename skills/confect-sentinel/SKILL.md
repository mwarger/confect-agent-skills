---
name: confect-sentinel
description: "Set up Sentinel production error capture in your own Convex deployment for a Confect app (Effect on Convex, a confect/ directory)."
---

<!-- GENERATED from convex-agents content/capabilities/sentinel.json — do not edit by hand. -->
<!-- Adapted to Confect. The component is plain Convex; app code that calls it uses the raw ctx. -->

# Capture production errors in your own deployment

Install `@convex-dev/sentinel` to capture production errors (server function failures, client JS/React crashes, OCC and scale signals) into a table in the user's OWN deployment, redacted at write time, then react to new ones. Data never leaves the user's deployment.

Before you write app code that calls the component, read `confect/references/confect-patterns.md`. Example E6 shows component calls. The Convex guidelines stay the source for platform rules.

## Workflow

1. Install the component: `app.use(sentinel)` in `convex/convex.config.ts`. You own that file. Confect codegen does not write it.
2. Wire the client SDK: a React error boundary plus `window.onerror`/`unhandledrejection` and breadcrumbs. This is client code. It does not use Confect.
3. Redaction runs at write time and is on by default (default-deny on secret key names and value patterns).
4. Read recent errors with the Convex CLI (`convex data`, `run-once-query`); react to new ones via the monitor's `prod_error` event (see confect-monitor).
5. Optionally enable the self-healing cron: `triage` classifies each error and, for recurring non-transient ones, hands it to ai-runner to open a fix PR (see confect-self-heal).

## Rules

- Redaction is mandatory and on by default — never store raw secrets; the agent's reads reach the model provider.
- Data stays in the user's deployment; never send it to a third party.
- Sample and cap to control volume and cost.
- Capturing PROD errors needs a deployed cloud app; install works anonymously.
- Confect does not wrap component clients. When app code must call a Sentinel function, take the raw ctx service (`MutationCtx`, `QueryCtx`, or `ActionCtx` from `confect/_generated/services`) and wrap the call in `Effect.promise`. Expose it to clients only through your own spec functions.
- A typed error declared in a spec's `error` schema is part of the API contract. An unexpected failure (a defect) is the production error to triage.
