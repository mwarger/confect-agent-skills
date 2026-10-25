---
name: confect-create-component
description:
  Builds reusable Convex components with isolated tables and app-facing Confect
  wrappers. Use in a Confect project (a confect/ directory) for new components,
  reusable backend modules, integrations, or component boundary work.
---

# Confect Create Component

Create reusable Convex components with clear boundaries and a small app-facing
API. The component is plain Convex code. The app side is Confect code.

Confect mounts and calls components. Confect does not build components.
Codegen reads the components that `convex/convex.config.ts` installs. It does
not read or change component code. Write the component code in plain Convex,
in its own directory. Write the app wrappers as Confect specs and impls.

For Confect patterns, read `references/confect-patterns.md` in the `confect`
skill. For Convex platform rules, read `convex/_generated/ai/guidelines.md` in
the app. If that file does not exist, read the Convex docs. Do not run
`npx convex ai-files install`.

## When to Use

- Creating a new Convex component in an existing Confect app
- Extracting reusable backend logic into a component
- Building a third-party integration that should own its own tables and
  workflows
- Packaging Convex functionality for reuse across multiple apps

## When Not to Use

- One-off business logic that belongs in the main app
- Thin utilities that do not need Convex tables or functions
- App-level orchestration that should stay in `confect/`
- Cases where a normal TypeScript library is enough

## Workflow

1. Ask the user what they are building and what the end goal is. If the repo
   already makes the answer obvious, say so and confirm before proceeding.
2. Choose the shape using the decision tree below and read the matching
   reference file.
3. Decide whether a component is justified. Prefer normal app code or a regular
   library if the feature does not need isolated tables, backend functions, or
   reusable persistent state.
4. Make a short plan for:
   - what tables the component owns
   - what public functions it exposes
   - what data must be passed in from the app (auth, env vars, parent IDs)
   - what stays in the app as Confect wrappers or HTTP mounts
5. Create the component structure with `convex.config.ts`, `schema.ts`, and
   function files. Write them in plain Convex (`v.*` validators, `ctx.db`).
   Do not use `@confect/*` in component code, because Confect codegen does
   not read component code.
6. Implement functions using the component's own `./_generated/server` imports,
   not the app's generated files.
7. Wire the component into the app with `app.use(...)`. If the app does not
   already have `convex/convex.config.ts`, create it.
8. Call the component from the app through `components.<name>`. Import
   `components` from `confect/_generated/components`, not from
   `convex/_generated/api`. Get the raw Convex ctx with `yield* QueryCtx`,
   `yield* MutationCtx`, or `yield* ActionCtx`. Call `ctx.runQuery`,
   `ctx.runMutation`, or `ctx.runAction` inside `Effect.promise`. Confect does
   not wrap component calls.
9. If React clients, HTTP callers, or public APIs need access, create Confect
   wrapper functions (a spec and an impl) in the app instead of exposing
   component functions directly.
10. Run the validation steps below and fix codegen, type, or boundary issues
    before finishing.

## Choose the Shape

Ask the user, then pick one path:

| Goal                                              | Shape            | Reference                           |
| ------------------------------------------------- | ---------------- | ----------------------------------- |
| Component for this app only                       | Local            | `references/local-components.md`    |
| Publish or share across apps                      | Packaged         | `references/packaged-components.md` |
| User explicitly needs local + shared library code | Hybrid           | `references/hybrid-components.md`   |
| Not sure                                          | Default to local | `references/local-components.md`    |

Read exactly one reference file before proceeding.

## Default Approach

Unless the user explicitly wants an npm package, default to a local component:

- Put it under `convex/components/<componentName>/`
- Define it with `defineComponent(...)` in its own `convex.config.ts`
- Install it from the app's `convex/convex.config.ts` with `app.use(...)`. You
  own that file. Confect does not change it.
- Let `npx convex codegen --component-dir convex/components/<name>` generate
  the component's own `_generated/` files. Confect codegen needs
  `_generated/component.js` from that step to type the `components` registry.

## Component Skeleton

A minimal local component with a table and two functions, plus the app wiring.
The component files are plain Convex. The app wrapper is a Confect spec and
impl.

```ts
// convex/components/notifications/convex.config.ts
import { defineComponent } from "convex/server";

export default defineComponent("notifications");
```

```ts
// convex/components/notifications/schema.ts (plain Convex)
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  notifications: defineTable({
    userId: v.string(),
    message: v.string(),
    read: v.boolean(),
  }).index("by_user_read", ["userId", "read"]),
});
```

```ts
// convex/components/notifications/lib.ts (plain Convex)
import { v } from "convex/values";
import { mutation, query } from "./_generated/server.js";

export const send = mutation({
  args: { userId: v.string(), message: v.string() },
  returns: v.id("notifications"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("notifications", {
      userId: args.userId,
      message: args.message,
      read: false,
    });
  },
});

export const listUnread = query({
  args: { userId: v.string() },
  returns: v.array(
    v.object({
      _id: v.id("notifications"),
      _creationTime: v.number(),
      userId: v.string(),
      message: v.string(),
      read: v.boolean(),
    }),
  ),
  handler: async (ctx, args) => {
    return await ctx.db
      .query("notifications")
      .withIndex("by_user_read", (q) =>
        q.eq("userId", args.userId).eq("read", false),
      )
      .collect();
  },
});
```

```ts
// convex/convex.config.ts
import { defineApp } from "convex/server";
import notifications from "./components/notifications/convex.config.js";

const app = defineApp();
app.use(notifications);

export default app;
```

```ts
// confect/notifications.spec.ts  (app-side wrapper: the spec)
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

export class Unauthenticated extends Schema.TaggedError<Unauthenticated>()(
  "Unauthenticated",
  {},
) {}

// The component returns ids as plain strings, so the wrapper uses Schema.String.
const Notification = Schema.Struct({
  _id: Schema.String,
  _creationTime: Schema.Finite,
  userId: Schema.String,
  message: Schema.String,
  read: Schema.Boolean,
});

export default GroupSpec.make()
  .addFunction(
    FunctionSpec.publicMutation({
      name: "sendNotification",
      args: () => ({ message: Schema.String }),
      returns: () => Schema.Null,
      error: () => Unauthenticated,
    }),
  )
  .addFunction(
    FunctionSpec.publicQuery({
      name: "myUnread",
      returns: () => Schema.Array(Notification),
      error: () => Unauthenticated,
    }),
  );
```

```ts
// confect/notifications.impl.ts  (app-side wrapper: the impl)
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import { components } from "./_generated/components";
import databaseSchema from "./_generated/schema";
import { Auth, MutationCtx, QueryCtx } from "./_generated/services";
import notifications, { Unauthenticated } from "./notifications.spec";

const sendNotification = FunctionImpl.make(
  databaseSchema,
  notifications,
  "sendNotification",
  ({ message }) =>
    Effect.gen(function* () {
      const auth = yield* Auth;
      const identity = yield* auth.getUserIdentity.pipe(
        Effect.catchTag("NoUserIdentityFoundError", () =>
          Effect.fail(new Unauthenticated()),
        ),
      );

      // Confect does not wrap component calls. Use the raw ctx.
      const ctx = yield* MutationCtx;
      yield* Effect.promise(() =>
        ctx.runMutation(components.notifications.lib.send, {
          userId: identity.tokenIdentifier,
          message,
        }),
      );
      return null;
    }),
);

const myUnread = FunctionImpl.make(databaseSchema, notifications, "myUnread", () =>
  Effect.gen(function* () {
    const auth = yield* Auth;
    const identity = yield* auth.getUserIdentity.pipe(
      Effect.catchTag("NoUserIdentityFoundError", () =>
        Effect.fail(new Unauthenticated()),
      ),
    );

    const ctx = yield* QueryCtx;
    return yield* Effect.promise(() =>
      ctx.runQuery(components.notifications.lib.listUnread, {
        userId: identity.tokenIdentifier,
      }),
    );
  }),
);

export default GroupImpl.make(databaseSchema, notifications).pipe(
  Layer.provide(Layer.mergeAll(sendNotification, myUnread)),
  GroupImpl.finalize,
);
```

Note the reference path shape: a function in
`convex/components/notifications/lib.ts` is called as
`components.notifications.lib.send` from the app.

The component write is part of the calling mutation's transaction. Do the auth
check in the wrapper before you call the component. Clients call the wrapper as
`refs.public.notifications.sendNotification` (Convex name
`notifications:sendNotification`).

## Critical Rules

- Keep authentication in the app, because `ctx.auth` is not available inside
  components. Read the identity with the Confect `Auth` service in the wrapper.
  Use `identity.tokenIdentifier` as the user key.
- Keep environment access in the app, because component functions cannot read
  `process.env`. Read variables with `Config` in the wrapper.
- Pass parent app IDs across the boundary as strings, because `Id` types become
  plain strings in the app-facing `ComponentApi`.
- Do not use `v.id("parentTable")` for app-owned tables inside component args or
  schema, because the component has no access to the app's table namespace. Do
  not use the Confect `Id("parentTable")` schema there either.
- Import `query`, `mutation`, and `action` from the component's own
  `./_generated/server`, not the app's generated files. Do not import from
  `confect/` or `@confect/*` in component code.
- Do not expose component functions directly to clients. Create Confect
  wrappers (a spec and an impl) when client access is needed, because
  components are internal and need auth/env wiring the app provides.
- If the component defines HTTP handlers, mount the routes in the app's
  `confect/http.ts`, because components cannot register their own HTTP routes.
  Confect writes `convex/http.ts`. Do not edit that file. For a plain Convex
  handler, use `http.route({ ..., handler: httpActionGeneric(...) })` on the
  router that `HttpRouter.make` returns.
- If the component needs pagination, use `paginator` from `convex-helpers`
  instead of built-in `.paginate()`, because `.paginate()` does not work across
  the component boundary. This rule applies to component code, which is plain
  Convex.
- Define indexes for queried fields instead of using Convex `.filter()` after a
  database query. The Confect query builder has no `.filter()`. Component code
  is plain Convex, so the rule still applies there.
- Add `args` and `returns` validators to all public component functions, because
  the component boundary requires explicit type contracts. Component functions
  use `v.*` validators. App wrappers use Effect Schema in the spec.
- Give each component a name that has only letters, digits, and underscores.
  The first character must be a letter or an underscore.
  Pass a unique `name` to `app.use` when you install the same component two
  times. Confect codegen rejects other names.

## Patterns

### Authentication and environment access

```ts
// Bad: component code cannot rely on app auth or env
// (inside the component, in plain Convex)
const identity = await ctx.auth.getUserIdentity();
const apiKey = process.env.OPENAI_API_KEY;
```

```ts
// Good: the wrapper resolves auth and env, then passes explicit values
import * as Config from "effect/Config";
import * as Effect from "effect/Effect";

import { components } from "./_generated/components";
import { ActionCtx, Auth } from "./_generated/services";
import { Unauthenticated } from "./translations.spec";

// Inside FunctionImpl.make(databaseSchema, translations, "translate", ...)
({ text }) =>
  Effect.gen(function* () {
    const auth = yield* Auth;
    const identity = yield* auth.getUserIdentity.pipe(
      Effect.catchTag("NoUserIdentityFoundError", () =>
        Effect.fail(new Unauthenticated()),
      ),
    );
    const apiKey = yield* Config.String("OPENAI_API_KEY");

    const ctx = yield* ActionCtx;
    return yield* Effect.promise(() =>
      ctx.runAction(components.translator.translate, {
        userId: identity.tokenIdentifier,
        apiKey,
        text,
      }),
    );
    // A missing variable is a defect, not a spec error.
  }).pipe(Effect.orDie);
```

### Client-facing API

```ts
// Bad: assuming a component function is directly callable by clients
// (components.* is not a Confect spec, so clients cannot call it)
export const send = components.notifications.send;
```

```ts
// Good: wrap it in an app function. The spec is in
// confect/notifications.spec.ts and the impl is in
// confect/notifications.impl.ts (see the skeleton above).
const sendNotification = FunctionImpl.make(
  databaseSchema,
  notifications,
  "sendNotification",
  ({ message }) =>
    Effect.gen(function* () {
      const auth = yield* Auth;
      const identity = yield* auth.getUserIdentity.pipe(
        Effect.catchTag("NoUserIdentityFoundError", () =>
          Effect.fail(new Unauthenticated()),
        ),
      );

      const ctx = yield* MutationCtx;
      yield* Effect.promise(() =>
        ctx.runMutation(components.notifications.lib.send, {
          userId: identity.tokenIdentifier,
          message,
        }),
      );
      return null;
    }),
);
```

### IDs across the boundary

```ts
// Bad: parent app table IDs are not valid component validators
// (in the component, which is plain Convex)
args: {
  userId: v.id("users"),
}
```

```ts
// Good: treat parent-owned IDs as strings at the boundary
// (in the component, which is plain Convex)
args: {
  userId: v.string(),
}

// In the app wrapper spec (Confect), use a string schema for the same field.
args: () => ({ userId: Schema.String })
```

### Advanced Patterns

For additional patterns including function handles for callbacks, deriving
validators from schema, static configuration with a globals table, and
class-based client wrappers, see `references/advanced-patterns.md`. The
component-side code in that file is plain Convex. The app-side code is Confect.

## Validation

Try validation in this order:

1. `npx convex codegen --component-dir convex/components/<name>`
2. `npx confect codegen`
3. `npx convex codegen`
4. `npx tsc --noEmit`
5. `npx convex dev` (with `npx confect dev` in a second terminal)

Important:

- Fresh repos may fail these commands until `CONVEX_DEPLOYMENT` is configured.
- Until codegen runs, component-local `./_generated/*` imports and app-side
  `components.<name>...` references will not typecheck. Run the component
  codegen (step 1) before `npx confect codegen`, because the Confect
  `components` registry imports `<component dir>/_generated/component.js`.
- Confect codegen does not write `convex/_generated/`. Run the app-level
  `npx convex codegen` (step 3) before `tsc`, or `tsc` can read a stale or
  missing `convex/_generated`.
- Codegen success alone is not enough. Gate on `tsc --noEmit`.
- Do not edit `confect/_generated/` or the generated files in `convex/`.
- If validation blocks on Convex login or deployment setup, stop and ask the
  user for that exact step instead of guessing.

## Reference Files

Read exactly one of these after the user confirms the goal:

- `references/local-components.md`
- `references/packaged-components.md`
- `references/hybrid-components.md`

Official docs:
[Authoring Components](https://docs.convex.dev/components/authoring)

## Checklist

- [ ] Asked the user what they want to build and confirmed the shape
- [ ] Read the matching reference file
- [ ] Confirmed a component is the right abstraction
- [ ] Planned tables, public API, boundaries, and app wrappers
- [ ] Component lives under `convex/components/<name>/` (or package layout if
      publishing)
- [ ] Component imports from its own `./_generated/server`
- [ ] Component code is plain Convex (no `@confect/*` imports)
- [ ] Auth, env access, and HTTP routes stay in the app (`Auth`, `Config`, and
      `confect/http.ts`)
- [ ] App wrappers are Confect specs and impls that call `components` from
      `confect/_generated/components` through the raw ctx
- [ ] The wrapper does the auth check before the component call
- [ ] Parent app IDs cross the boundary as `v.string()` (`Schema.String` in the
      wrapper spec)
- [ ] Public functions have `args` and `returns` validators
- [ ] Ran component codegen, `npx confect codegen`, `npx convex codegen`, and
      `npx tsc --noEmit`
- [ ] Ran `npx convex dev` and fixed codegen or type issues
