# Ledger: convex-billing (SKILL.md; no references/ directory)

Source: 69 lines. Code skill (not platform only).

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter name `convex-billing` | Update to `confect-billing` | Confect naming. Directory name kept. |
| Frontmatter description | Update | Mention Confect and the `confect/` trigger. |
| GENERATED comment | Keep | Unchanged upstream line, keeps syncs small. |
| Title `# Add billing / payments` | Keep | Platform-neutral. |
| Intro paragraph | Keep | Same architecture holds in Confect. |
| (new) paragraph on raw ctx and pointers | Add | Points to confect-patterns.md (E5, E6, E12, E13) and references/guidelines.md. Replaces the Convex guidelines pointer. |
| Step 1 npm install | Keep | Platform command. User instruction forbids installs for the agent only, not the skill text. |
| Step 2 convex.config.ts + code | Keep code, add one sentence | Confect says the user owns this file. |
| Step 3 env keys | Keep | Platform rule. |
| Step 4 http.ts + `httpRouter()` code | Update | Becomes `confect/http.ts` with `HttpRouter.make(Layer.empty)`. `registerRoutes` stays as an escape hatch on the returned router. `components` from `confect/_generated/components`. |
| Step 5 billing.ts: `action`, `query`, `v.*` | Update | Split into spec and impl. `publicAction`, `publicQuery`, Schema returns. |
| `ctx.auth.getUserIdentity()` + `throw new Error` | Update | `Auth` service, `NoUserIdentityFoundError` mapped to typed `NotSignedIn` error. |
| `process.env.SITE_URL ?? ...` | Update | `Config.String` with `Config.withDefault`. |
| `identity.subject` as userId | Update | `tokenIdentifier` per patterns file rule (never `subject` alone). Open question below. |
| `stripeClient` calls with ctx | Update | Raw `ActionCtx` in `Effect.promise`. |
| `ctx.runQuery(components.stripe.public...)` | Update | Raw `QueryCtx` in `Effect.promise`. Component refs come from Confect registry. |
| `isSubscribed` not signed in returns false | Keep behavior | Mapped with `catchTag` to `null`. |
| Step 6 `npx convex dev --once` and verify line | Keep, add | Add `npx confect codegen` first and `tsc --noEmit` after. |
| Step 7 Stripe dashboard webhook | Keep | Platform fact. |
| Rule 1 use component, no manual constructEvent | Keep | Unchanged. |
| Rule 2 keys in Convex env | Keep, add | Add `Config` for other settings. |
| Rule 3 gate on server state | Keep | Unchanged. |
| Rule 4 `.js` extension in convex.config | Keep | Unchanged. |
| (new) rules: components ref, tokenIdentifier, raw ctx, typed errors | Add | Confect-specific, from patterns file. |

Counts: kept 12, updated 11, added 3, removed 0.

Open questions:
- @convex-dev/stripe is not installed in the verified sources. Its API names (`StripeSubscriptions`, `registerRoutes`, `getOrCreateCustomer`, `createCheckoutSession`, `listSubscriptionsByUserId`) are copied from upstream and not compiled.
- `registerRoutes` on the Confect router: relies on the router being a Convex `HttpRouter` (patterns E13 uses `http.route`). Not compiled with this component.
- `tokenIdentifier` replaces `subject` as the Stripe userId. This breaks existing customer data keyed by `subject`. Confirm.
- `identity.email` and `identity.name` may be `undefined`. Under `exactOptionalPropertyTypes` the component types may reject that.
