---
name: confect-migrate
description: "Migrate a Confect schema (confect/tables/) + backfill data on a deployed app using @convex-dev/migrations. Use for a project with a confect/ directory."
---

<!-- GENERATED from convex-agents content/capabilities/migrate.json — do not edit by hand. -->

# Migrate the schema / data on a live app

Change a deployed schema without breaking existing data: stage the schema change, install @convex-dev/migrations, write a backfill that makes old rows valid, run it, and verify before tightening the validator.

Before you change a Confect schema, read `confect/references/confect-patterns.md` (the confect skill). The platform rules (limits, indexes, transactions) are the same as in plain Convex. The platform behavior is the same as in plain Convex: a push checks every existing document against the new schema and fails if one does not match. Confect changes where you write the schema and the backfill.

## Where the code goes

- The schema is the table files in `confect/tables/`. Codegen writes `convex/schema.ts`. Do not edit `convex/schema.ts`.
- A field is optional with `Schema.optionalKey(...)`. A field that can hold `null` uses `Schema.NullOr(...)`. Required is the plain schema.
- Mount the component in `convex/convex.config.ts` (you own that file): `app.use(migrations)` with `migrations` from `@convex-dev/migrations/convex.config`. Get the component reference from `confect/_generated/components`, not from `convex/_generated/api`.
- The `@convex-dev/migrations` client takes the raw Convex ctx and plain Convex functions. Write the migration in `confect/migrations.ts` with the library's `define` and `runner` helpers. Name the functions in `confect/migrations.spec.ts` with `FunctionSpec.convexInternalMutation<typeof backfill>()("backfill")`, and use a type-only import there. The impl passes the function value through. See E12 in `confect-patterns.md`.
- A backfill reads and writes rows with the raw Convex `ctx.db`. Confect does not decode or encode those rows. The row type is the Convex data model type. Keep the new field name and shape equal to the Confect table schema.
- After each schema change, run `npx confect codegen`, then `tsc --noEmit`, then deploy.

## Workflow

1. Make the new field optional first (so deploy doesn't reject existing rows): change the table file in `confect/tables/` to `Schema.optionalKey(...)`, run `npx confect codegen` and `tsc --noEmit`, and deploy.
2. Install @convex-dev/migrations; write a migration that backfills/transforms existing rows.
3. Run the migration; verify all rows are valid.
4. Tighten the validator (make the field required) once the backfill is complete: change the table file to the plain schema, run `npx confect codegen` and `tsc --noEmit`, and deploy. Fix the impls that read the field, so that they no longer handle the missing case.

## Rules

- Never tighten a validator before the backfill completes — it rejects existing rows and breaks the live app.
- Add new fields as optional first, migrate, then require.
- Verify row counts before and after.
- Do not edit `convex/schema.ts` or `confect/_generated/`. Change the table files and run codegen.
- A Confect index change follows the same platform rule as Convex: a new index on a large table blocks the deploy until backfill completes. Confect 10.0.0 has no staged indexes, so plan for the block.
- Write the backfill as a plain Convex function (the `@convex-dev/migrations` helpers). Do not write it as a Confect `FunctionImpl` that loads a whole table.
