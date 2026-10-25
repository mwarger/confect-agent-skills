# Advanced Component Patterns

Additional patterns for Convex components that go beyond the basics covered in
the main skill file. Component-side code is plain Convex. App-side code is
Confect.

## Function Handles for callbacks

When the app needs to pass a callback function to the component, use function
handles. This is common for components that run app-defined logic on a schedule
or in a workflow.

Confect does not wrap function handles. Mint the handle with
`createFunctionHandle` from `convex/server`. Pass `Ref.getFunctionReference`
of the Confect ref. The function that the component calls back must be a plain
Convex function. Confect does not decode or encode its args. Declare it in the
spec with a type-only import and `FunctionSpec.convexInternalMutation`.

```ts
// confect/jobs.ts: a plain Convex function that the component calls back
import { internalMutationGeneric } from "convex/server";
import { v } from "convex/values";

export const processItem = internalMutationGeneric({
  args: {},
  returns: v.null(),
  handler: async () => {
    return null;
  },
});
```

```ts
// confect/jobs.spec.ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

import type { processItem } from "./jobs";

export default GroupSpec.make()
  // Type-only import: the spec ships to the client.
  .addFunction(
    FunctionSpec.convexInternalMutation<typeof processItem>()("processItem"),
  )
  .addFunction(
    FunctionSpec.publicMutation({
      name: "startJob",
      returns: () => Schema.Null,
    }),
  );
```

```ts
// confect/jobs.impl.ts (excerpt): App side: create a handle and pass it to the component
import { Ref } from "@confect/core";
import { FunctionImpl, GroupImpl } from "@confect/server";
import { createFunctionHandle } from "convex/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import { components } from "./_generated/components";
import refs from "./_generated/refs";
import databaseSchema from "./_generated/schema";
import { MutationCtx } from "./_generated/services";
import { processItem } from "./jobs";
import jobs from "./jobs.spec";

// A plain Convex function's impl is the Convex function value itself.
const processItemImpl = FunctionImpl.make(
  databaseSchema,
  jobs,
  "processItem",
  processItem,
);

const startJob = FunctionImpl.make(databaseSchema, jobs, "startJob", () =>
  Effect.gen(function* () {
    const ctx = yield* MutationCtx;
    const handle = yield* Effect.promise(() =>
      createFunctionHandle(
        Ref.getFunctionReference(refs.internal.jobs.processItem),
      ),
    );
    yield* Effect.promise(() =>
      ctx.runMutation(components.workpool.enqueue, { callback: handle }),
    );
    return null;
  }),
);

export default GroupImpl.make(databaseSchema, jobs).pipe(
  Layer.provide(Layer.mergeAll(processItemImpl, startJob)),
  GroupImpl.finalize,
);
```

```ts
// Component side (plain Convex): accept and invoke the handle
import { v } from "convex/values";
import type { FunctionHandle } from "convex/server";
import { mutation } from "./_generated/server.js";

export const enqueue = mutation({
  args: { callback: v.string() },
  handler: async (ctx, args) => {
    const handle = args.callback as FunctionHandle<"mutation">;
    await ctx.scheduler.runAfter(0, handle, {});
  },
});
```

## Deriving validators from schema

Instead of manually repeating field types in return validators, extend the
schema validator:

This code is in the component, so it is plain Convex. Confect tables do not
give a component its validators. Keep the component schema in plain
`defineSchema`.

```ts
import { v } from "convex/values";
import schema from "./schema.js";

const vNotification = schema.doc("notifications").omit("userId").extend({
  user: v.string(),
});

export const getNotification = internalQuery({
  args: { id: schema.id("notifications") },
  returns: v.nullable(vNotification),
  handler: async (ctx, args) => {
    const notification = await ctx.db.get("notifications", args.id);
    if (!notification) return null;
    const { userId, ...rest } = notification;
    const user = await ctx.db.get("users", userId);
    return {
      ...rest,
      user: user?.name ?? "Unknown",
    };
  },
});
```

## Static configuration with a globals table

A common pattern for component configuration is a single-document "globals"
table:

The globals table is part of the component, so it is plain Convex.

```ts
// schema.ts (component)
export default defineSchema({
  globals: defineTable({
    maxRetries: v.number(),
    webhookUrl: v.optional(v.string()),
  }),
  // ... other tables
});
```

```ts
// lib.ts (component)
export const configure = mutation({
  args: { maxRetries: v.number(), webhookUrl: v.optional(v.string()) },
  returns: v.null(),
  handler: async (ctx, args) => {
    const existing = await ctx.db.query("globals").first();
    if (existing) {
      await ctx.db.patch(existing._id, args);
    } else {
      await ctx.db.insert("globals", args);
    }
    return null;
  },
});
```

## Class-based client wrappers

For components with many functions or configuration options, a class-based
client provides a cleaner API. This pattern is common in published components.

The client class is plain TypeScript. It takes any ctx that has `runMutation`.
In a Confect app, pass the raw ctx from `yield* MutationCtx`.

```ts
// src/client/index.ts
import type { GenericMutationCtx, GenericDataModel } from "convex/server";
import type { ComponentApi } from "../component/_generated/component.js";

type MutationCtx = Pick<GenericMutationCtx<GenericDataModel>, "runMutation">;

export class Notifications {
  constructor(
    private component: ComponentApi,
    private options?: { defaultChannel?: string },
  ) {}

  async send(ctx: MutationCtx, args: { userId: string; message: string }) {
    return await ctx.runMutation(this.component.lib.send, {
      ...args,
      channel: this.options?.defaultChannel ?? "default",
    });
  }
}
```

```ts
// App usage: confect/notifications.impl.ts (excerpt)
import { Notifications } from "@convex-dev/notifications";
import { FunctionImpl } from "@confect/server";
import * as Effect from "effect/Effect";

import { components } from "./_generated/components";
import databaseSchema from "./_generated/schema";
import { Auth, MutationCtx } from "./_generated/services";
import notificationsGroup, { Unauthenticated } from "./notifications.spec";

const notifications = new Notifications(components.notifications, {
  defaultChannel: "alerts",
});

// Use the Confect components registry, not convex/_generated/api.
export const send = FunctionImpl.make(
  databaseSchema,
  notificationsGroup,
  "send",
  ({ message }) =>
    Effect.gen(function* () {
      const auth = yield* Auth;
      const identity = yield* auth.getUserIdentity.pipe(
        Effect.catchTag("NoUserIdentityFoundError", () =>
          Effect.fail(new Unauthenticated()),
        ),
      );

      // Component clients take the raw Convex ctx.
      const ctx = yield* MutationCtx;
      yield* Effect.promise(() =>
        notifications.send(ctx, {
          userId: identity.tokenIdentifier,
          message,
        }),
      );
      return null;
    }),
);
```
