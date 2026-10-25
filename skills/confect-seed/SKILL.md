---
name: confect-seed
description: "Seed or import data into the Convex database of a Confect app (Effect on Convex, a confect/ directory). Use an internal mutation spec and impl, or npx convex import."
---

<!-- GENERATED from convex-agents content/capabilities/seed.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Seed / import data

Populate tables via an internal mutation seed function (re-runnable) or `npx convex import`, matching the schema.

Before you write the seed function, read `confect/references/confect-patterns.md`. Examples E2 and E4 show specs and writes. The Convex guidelines stay the source for platform rules.

## Workflow

1. For fixtures: write an internal mutation that inserts sample rows. Declare it with `FunctionSpec.internalMutation` in a spec file. Implement it with `FunctionImpl.make` and `DatabaseWriter`. Run `npx confect codegen`. Then run it with `npx convex run <module>:<name>`. The module is the file path with slashes, not dots. A group `billing.invoices` gives `billing/invoices:run`. The example below is in `confect/seed.spec.ts`, so the command is `npx convex run seed:run`.
2. For bulk import: shape the data to the schema and use `npx convex import`. Import writes raw Convex values. Confect does not decode them. Use the Encoded form of each field. For example, a field that uses `Schema.DateTimeUtcFromMillis` takes a number.
3. Make seeding idempotent (clear-then-insert or upsert) so re-running is safe.
4. Verify row counts.

```ts
// confect/seed.spec.ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

// Good: internal, so the public Internet cannot call it.
export default GroupSpec.make().addFunction(
  FunctionSpec.internalMutation({
    name: "run",
    returns: () => Schema.Null,
  }),
);
```

```ts
// Bad: a public mutation can be called by anyone on the Internet.
FunctionSpec.publicMutation({ name: "run", returns: () => Schema.Null });
```

```ts
// Bad: collect() reads every row. It breaks the read limits on a large table.
const everyNote = yield* reader.table("notes").index("by_creation_time").collect();
```

```ts
// confect/seed.impl.ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Duration from "effect/Duration";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import refs from "./_generated/refs";
import databaseSchema from "./_generated/schema";
import {
  DatabaseReader,
  DatabaseWriter,
  Scheduler,
} from "./_generated/services";
import seed from "./seed.spec";

const BATCH_SIZE = 100;

const run = FunctionImpl.make(databaseSchema, seed, "run", () =>
  Effect.gen(function* () {
    const reader = yield* DatabaseReader;
    const writer = yield* DatabaseWriter;
    const scheduler = yield* Scheduler;

    // Good: clear in a bounded batch. Schedule a continuation when the batch is full.
    const existing = yield* reader
      .table("notes")
      .index("by_creation_time")
      .take(BATCH_SIZE);
    for (const note of existing) {
      yield* writer.table("notes").delete(note._id);
    }
    if (existing.length === BATCH_SIZE) {
      yield* scheduler.runAfter(Duration.zero, refs.internal.seed.run);
      return null;
    }

    // Good: upsert the parent row. Insert it only when no row matches.
    const author = yield* reader
      .table("users")
      .get("by_tokenIdentifier", "seed|author")
      .pipe(
        Effect.catchTag("GetByIndexFailure", () =>
          writer
            .table("users")
            .insert({ tokenIdentifier: "seed|author", name: "Seed Author" })
            .pipe(Effect.flatMap((authorId) => reader.table("users").get(authorId))),
        ),
      );

    // Good: a fixture row that matches the table schema, with its required authorId.
    yield* writer.table("notes").insert({ authorId: author._id, text: "First note" });
    return null;
  }).pipe(Effect.orDie),
);

export default GroupImpl.make(databaseSchema, seed).pipe(
  Layer.provide(run),
  GroupImpl.finalize,
);
```

## Rules

- Seed via an internal mutation (`FunctionSpec.internalMutation`) or convex import, matching the table schemas.
- Make seeding idempotent.
- Never seed secrets/PII into a shared deployment.
- A seed run is one transaction. Keep each run within the Convex write limits. For more rows, do one batch and schedule the rest with `Scheduler.runAfter` on an internal ref.
- To upsert, read with `reader.table(t).get(indexName, ...values)`. It fails with `GetByIndexFailure` when no row matches. Recover with `Effect.catchTag("GetByIndexFailure", ...)` and insert.
