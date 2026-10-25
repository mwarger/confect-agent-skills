---
name: confect-billing
description: "Add Stripe billing/payments to a Confect app (Effect on Convex) via @convex-dev/stripe (checkout + webhook + gating). Use when a project has a confect/ directory."
---

<!-- GENERATED from convex-agents content/capabilities/billing.json — do not edit by hand. -->

# Add billing / payments

Wire Stripe to Confect using @convex-dev/stripe: a checkout action, an httpAction webhook registered by the component (signature-verified automatically), subscription state stored in the component's tables, and server-side gating via a query.

Confect does not wrap component clients. Pass the raw ctx service (`ActionCtx`, `QueryCtx`) to them and wrap each call in `Effect.promise`. For the patterns used here, read the confect skill's `references/confect-patterns.md` (E5, E6, E12, E13). For platform rules, read its `references/guidelines.md`.

## Workflow

1. Install the component: `npm install @convex-dev/stripe`.
2. Create `convex/convex.config.ts`. You own this file in Confect. Codegen does not write it.
   ```ts
   import { defineApp } from 'convex/server';
   import stripe from '@convex-dev/stripe/convex.config.js';
   const app = defineApp();
   app.use(stripe);
   export default app;
   ```
3. Store Stripe keys in Convex env (use the `env` micro power): `STRIPE_SECRET_KEY` (sk_test_… / sk_live_…) and `STRIPE_WEBHOOK_SECRET` (whsec_…).
4. Create `confect/http.ts` to register the webhook route (the component handles signature verification automatically). `HttpRouter.make` returns a Convex router. Call `registerRoutes` on it. Exact paths match before the Confect catch-all route.
   ```ts
   import { HttpRouter as ConfectHttpRouter } from '@confect/server';
   import * as Layer from 'effect/Layer';
   import { registerRoutes } from '@convex-dev/stripe';
   import { components } from './_generated/components';
   // Use Layer.empty when the app has no Effect HTTP routes.
   const http = ConfectHttpRouter.make(Layer.empty);
   // Escape hatch: a component route on the plain Convex router.
   registerRoutes(http, components.stripe, { webhookPath: '/stripe/webhook' });
   export default http;
   ```
5. Create the billing group: a spec, then an impl. The checkout function is a public action. The gate is a public query.

   `confect/billing.spec.ts`
   ```ts
   import { FunctionSpec, GroupSpec } from '@confect/core';
   import * as Schema from 'effect/Schema';

   export class NotSignedIn extends Schema.TaggedError<NotSignedIn>()('NotSignedIn', {}) {}

   export default GroupSpec.make()
     .addFunction(
       FunctionSpec.publicAction({
         name: 'createSubscriptionCheckout',
         args: () => ({ priceId: Schema.String }),
         returns: () => Schema.Struct({ sessionId: Schema.String, url: Schema.NullOr(Schema.String) }),
         error: () => NotSignedIn,
       }),
     )
     .addFunction(
       FunctionSpec.publicQuery({
         name: 'isSubscribed',
         returns: () => Schema.Boolean,
       }),
     );
   ```

   `confect/billing.impl.ts`
   ```ts
   import { FunctionImpl, GroupImpl } from '@confect/server';
   import { StripeSubscriptions } from '@convex-dev/stripe';
   import * as Config from 'effect/Config';
   import * as Effect from 'effect/Effect';
   import * as Layer from 'effect/Layer';

   import { components } from './_generated/components';
   import databaseSchema from './_generated/schema';
   import { ActionCtx, Auth, QueryCtx } from './_generated/services';
   import billing, { NotSignedIn } from './billing.spec';

   const stripeClient = new StripeSubscriptions(components.stripe, {});

   const createSubscriptionCheckout = FunctionImpl.make(databaseSchema, billing, 'createSubscriptionCheckout', ({ priceId }) =>
     Effect.gen(function* () {
       const auth = yield* Auth;
       const ctx = yield* ActionCtx;
       const identity = yield* auth.getUserIdentity.pipe(
         Effect.catchTag('NoUserIdentityFoundError', () => Effect.fail(new NotSignedIn())),
       );
       const siteUrl = yield* Config.String('SITE_URL').pipe(Config.withDefault('http://localhost:3000'), Effect.orDie);
       // The component client takes the raw ctx. Wrap each call in Effect.promise.
       const customer = yield* Effect.promise(() =>
         stripeClient.getOrCreateCustomer(ctx, { userId: identity.subject, email: identity.email, name: identity.name }),
       );
       return yield* Effect.promise(() =>
         stripeClient.createCheckoutSession(ctx, {
           priceId,
           customerId: customer.customerId,
           mode: 'subscription',
           successUrl: `${siteUrl}/?success=true`,
           cancelUrl: `${siteUrl}/?canceled=true`,
           subscriptionMetadata: { userId: identity.subject },
         }),
       );
     }),
   );

   const isSubscribed = FunctionImpl.make(databaseSchema, billing, 'isSubscribed', () =>
     Effect.gen(function* () {
       const auth = yield* Auth;
       const ctx = yield* QueryCtx;
       // Not signed in means not subscribed. Return false, not an error.
       const identity = yield* auth.getUserIdentity.pipe(
         Effect.catchTag('NoUserIdentityFoundError', () => Effect.succeed(null)),
       );
       if (identity === null) return false;
       const subscriptions = yield* Effect.promise(() =>
         ctx.runQuery(components.stripe.public.listSubscriptionsByUserId, { userId: identity.subject }),
       );
       return subscriptions.some((sub) => sub.status === 'active' || sub.status === 'trialing');
     }),
   );

   export default GroupImpl.make(databaseSchema, billing).pipe(
     Layer.provide(Layer.mergeAll(createSubscriptionCheckout, isSubscribed)),
     GroupImpl.finalize,
   );
   ```
6. Run `npx confect codegen`, then `npx convex dev --once` — it will install the component and push the functions. Verify output shows `✔ Installed component stripe.` Then run `tsc --noEmit`. Codegen success alone is not enough.
7. In Stripe Dashboard → Webhooks: add endpoint `https://<deployment>.convex.site/stripe/webhook`, subscribe to `checkout.session.completed`, `customer.subscription.*`, `invoice.*`, `payment_intent.*`. Copy the signing secret as `STRIPE_WEBHOOK_SECRET`.

## Rules

- Use @convex-dev/stripe (npm: @convex-dev/stripe@^0.1.4) — it handles webhook signature verification internally via registerRoutes; do NOT write a manual constructEvent webhook.
- Stripe keys live in Convex env (use the `env` micro power): STRIPE_SECRET_KEY and STRIPE_WEBHOOK_SECRET. Read other settings, such as SITE_URL, with `Config`.
- Gate on server-stored subscription state via isSubscribed query (reads component tables), not client claims.
- convex/convex.config.ts must import from '@convex-dev/stripe/convex.config.js' (not .ts) — the .js extension is required by the Convex bundler.
- Get the component reference from `confect/_generated/components`, not from `convex/_generated/api`.
- Key Stripe users by `identity.subject`, as upstream does. Do not change the key in an existing app: customers saved under the old key lose their link. A new app with more than one auth issuer may use `identity.tokenIdentifier` from the start. Never take a user id as an argument.
- Component clients take the raw ctx. Use `ActionCtx` or `QueryCtx`, and wrap each call in `Effect.promise`.
- Declare expected failures in the spec with an `error` schema. Fail with a `Schema.TaggedError`. Other failures are defects.
