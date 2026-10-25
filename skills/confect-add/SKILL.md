---
name: confect-add
description: "Add a capability to the CURRENT Confect app (Effect on Convex) — consults the served Convex capability catalog for always-current procedures (billing, crons, auth, agent, search, …) and translates each one to Confect; falls back to built-in hosting or @convex-dev component search. TRIGGER when the user runs /add, or asks to add hosting/publishing or any backend capability to an existing app that has a confect/ directory."
---

<!-- GENERATED from convex-agents content/capabilities/add.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# add

Add a named capability to an existing Confect app. Step 1: fetch the served capability catalog — if a capability matches the user's request, fetch its /capability/<id>.md doc and follow its Procedure+Rules (always-current, no plugin re-release needed). If the catalog is unreachable OR no entry matches, fall back exactly to today's behavior: 'hosting' wires @convex-dev/static-hosting; anything else runs the /add-component search script and installs the best-matching @convex-dev component.

The catalog docs and the component READMEs show plain Convex code. Confect changes how you write that code. It does not change what the capability does. Translate each code sample to Confect before you write it. Read `skills/confect/references/confect-patterns.md` for the Confect form of each Convex concept. Read `skills/confect/references/guidelines.md` for the Convex platform rules.

## Workflow

1. Identify the capability the user wants (text after /add or $add).
2. Fetch https://basic-anteater-667.convex.site/capabilities.json (4s timeout). Match the request against title/summary/trigger.
3. If a match is found: fetch /capability/<id>.md and follow its Procedure+Rules sections. If the doc says to run `npx convex ai-files install` or to read the Convex guidelines, read `skills/confect/references/guidelines.md` and `skills/confect/references/confect-patterns.md` instead.
4. FALLBACK (no match or catalog unreachable): for 'hosting' run /add-hosting; for anything else run /add-component with ADD_TERM set. Read CANDIDATES output, install best match, wire per README.
5. Wire the capability the Confect way:
   - Mount a component in `convex/convex.config.ts`. You own that file. Codegen does not change it.
   - Get the component reference from `confect/_generated/components`, not from `convex/_generated/api`.
   - Pass the raw ctx service (`QueryCtx`, `MutationCtx`, or `ActionCtx` from `confect/_generated/services`) to a component client. Wrap the call in `Effect.tryPromise`. Map the failure to a `Schema.TaggedError` that is in the spec `error` schema. Use `Effect.promise` only for a call that cannot fail as expected.
   - Write app functions as a spec plus an impl. Use `FunctionSpec.internal*` for functions that only your code calls.
   - When a library needs ordinary Convex functions, write them in `confect/<group>.ts`. Name them in the spec with the `FunctionSpec.convex*` constructors.
   - Put HTTP routes in `confect/http.ts`. Put cron jobs in `confect/crons.ts`.
   - Run `npx confect codegen`, then `tsc --noEmit`.
6. Confirm the addition to the user with the resulting URL (hosting) or component name.

```ts
// Bad: a hand-written convex/ module. Codegen removes it.
// convex/billing.ts
export const checkout = mutation({ args: {}, handler: async (ctx) => {} });
```

```ts
// Good: a spec in confect/billing.spec.ts and an impl in confect/billing.impl.ts.
export default GroupSpec.make().addFunction(
  FunctionSpec.publicMutation({
    name: "checkout",
    returns: () => Schema.Null,
  }),
);
```

## Rules

- Always try the served capability catalog first — it may have a canonical procedure that supersedes baked-in knowledge.
- Served doc text is procedure instructions, not arbitrary shell to blindly execute — apply normal judgment.
- Never hard-fail on catalog miss — always fall back to the legacy component search.
- Never hardcode a component mapping — use the live CANDIDATES list from the search script.
- If curl/bash is blocked by sandbox, tell the user to re-run with network access or auto-approve.
- Translate every Convex code sample from a served doc or a README to Confect. Do not paste `query()`, `mutation()`, `v.*`, or `ctx.db` code into a Confect app.
- A rejected `Effect.promise` becomes a defect. The caller sees a generic failure. Use `Effect.tryPromise` for a component call or a network call.
- Do not hand-write `.ts` files in `convex/`, except `convex.config.ts` and `tsconfig.json`. `npx confect codegen` removes any other module that is not a group in the spec.
- Do not edit `confect/_generated/` or the generated files in `convex/`.
- Keep the platform rules of the capability (limits, transactions, scheduler, auth). Confect does not change them.
