---
name: confect-env
description: "Set and wire Convex deployment env vars / secrets for a Confect app, and read them with Effect Config (Config.String, Config.Redacted)."
---

<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Manage env vars + secrets

Store secrets as Convex deployment env vars (npx convex env set), read them with Effect `Config` in actions, never commit them.

Read `references/confect-patterns.md` (example E10) in the confect skill before you write code. For the Convex platform rules, read the project's `convex/_generated/ai/guidelines.md` when that file exists (`npx convex ai-files install` creates it). Confect installs a `ConfigProvider` for the Convex runtime, so `Config` reads Convex env vars. Do not use `process.env` in Confect code. `CONVEX_SITE_URL` and `CONVEX_CLOUD_URL` are always set. Node actions (`GroupSpec.makeNode()`) use Effect's default provider.

## Workflow

1. `npx convex env set KEY value` (per deployment). This command is the same in Confect.
2. Read via `yield* Config.String("KEY")` inside actions (not queries/mutations). Use `Config.Redacted("KEY")` for secrets, so logs and `toString` hide the value.
3. Never hardcode or commit secrets; add to .env.local only for local.
4. Confirm it's set with `npx convex env list --names-only`, which lists names without printing the secret values.

```ts
// Bad: raw process.env in a Confect impl.
const apiKey = process.env.API_KEY;

// Good: Config reads the Convex env var. Redacted hides the secret.
const apiKey = yield* Config.Redacted("API_KEY");
const maxResults = yield* Config.Int("MAX_RESULTS").pipe(
  Config.withDefault(20),
);
```

## Rules

- Secrets live in Convex env vars, never in code or git.
- Read env vars with `Config` only in actions (a Node action group if the code needs Node), not queries/mutations. Do not use `process.env` or `env` from `_generated/server`.
- Different deployments need their own values.
- Check what is set with `npx convex env list --names-only`; never print secret values into the conversation.
- `Config.Array` and `Config.Record` read one delimited variable (for example `A=1,B=2`). The Convex provider cannot build a collection from several variables.
- Confect does not surface the typed app env (`defineApp({ env })`). Decode a value with `Config.schema` when you need a type.
- A missing variable fails the `Config` Effect. Handle it with `Effect.orDie` or map it to a declared error.
