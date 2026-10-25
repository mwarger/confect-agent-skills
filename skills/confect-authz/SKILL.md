---
name: confect-authz
description: "Audit and harden the authorization of a Confect app (Effect on Convex): identity-from-arg impersonation, missing per-document ownership checks, public functions that leak data by a client-supplied id, and writes into a parent/container the caller doesn't own. Scans specs and impls for the 4 shapes, applies the RequireUser middleware and an ownership check, verifies with confect codegen and tsc. TRIGGER on 'secure my app', 'audit auth/authz', 'who can access this data' in a project with a confect/ directory. SKIP when there is no confect/ directory."
---

<!-- GENERATED from convex-agents content/capabilities/convex-authz.json — do not edit by hand. -->

# Confect Authz Auditor/Hardener

A focused authz specialist, not a general reviewer: it finds and fixes the four shapes that account for the largest real-defect cluster measured against generated Convex backends (25 identity-from-arg + 13 missing-ownership-check + 6 PII-leak-by-argument = 44 of 214 confirmed defects, plus the parent-reference-on-write variant of the ownership shape that fixture measurement showed the 3-shape scan misses). The measurements come from plain Convex backends. The shapes are the same in Confect. It runs a deterministic scan first (objective, regex-based), then applies the canonical hardening pattern to every hit, then verifies with codegen and tsc. It does not re-derive the pattern. In Confect, the canonical pattern is the `RequireUser` middleware and the owned-document read from the Confect patterns file (E3 and E5).

Confect changes how you write the code. It does not change how Convex behaves. Every public function is still reachable from the Internet. The Confect patterns file is `skills/confect/references/confect-patterns.md` (the `confect` skill's `references/` directory). This skill has no `references/` directory of its own. Read the patterns file for the API names. Read the Convex platform rules in the project's `convex/_generated/ai/guidelines.md` when that file exists.

## Workflow

0. MANDATORY FIRST STEP — check the auth foundation exists before injecting any identity enforcement. Check two things.
   - (1) Is there a `confect/auth.ts` with a provider? Codegen writes `convex/auth.config.ts`. That file imports `confect/auth.ts` and re-exports it.
   - (2) Is there a users or identities table keyed to the auth identity, with an index on the key field? The key is `tokenIdentifier` or `subject`. `tokenIdentifier` is preferred. Any table name is fine.

   If a `subject`-keyed foundation exists, it counts as a working foundation. Do not treat it as missing. Report it as a finding: the guideline prefers `tokenIdentifier`.

   If EITHER (1) or (2) is missing, DO NOT add the `RequireUser` middleware or an ownership check. On a foundationless app `auth.getUserIdentity` always fails with `NoUserIdentityFoundError`. Enforcement is then non-functional: every call is refused, or the check is bypassed or compared against a non-identity field like an email string. A reviewer correctly flags that as a NEW authz defect, not a fix. Instead, on a foundationless app:
   - (a) For privileged/admin operations, convert the public spec to an internal spec. This removes public reachability entirely. It is safe and needs no auth foundation. Use these maps:
     - `FunctionSpec.publicQuery` to `internalQuery`, `publicMutation` to `internalMutation`, `publicAction` to `internalAction`.
     - The same for the paginated and Node forms.
     - The plain forms: `convexPublicQuery` to `convexInternalQuery`, `convexPublicMutation` to `convexInternalMutation`, `convexPublicAction` to `convexInternalAction`, `convexPublicNodeAction` to `convexInternalNodeAction`.
     - In `confect/<group>.ts`, the generic builders: `queryGeneric` to `internalQueryGeneric`, `mutationGeneric` to `internalMutationGeneric`, `actionGeneric` to `internalActionGeneric`.
     Then change each caller from `refs.public.<group>.<name>` to `refs.internal.<group>.<name>`, and run `npx confect codegen`.
   - (b) Tell the user: 'this app has no auth foundation; set up auth first (see the confect-auth skill and the confect skill's E14 and E5), then re-run this skill to add per-user ownership checks.'

   Do not run steps 1-3 below against public functions on a foundationless app beyond this internalize-and-defer move. Only when the foundation exists (both `confect/auth.ts` and a users table keyed to the identity are present) do you proceed to add the middleware and ownership checks in steps 1-3.
1. SCAN (deterministic, objective-first): for every `confect/**/*.spec.ts`, `confect/**/*.impl.ts`, `confect/http*` file and plain `confect/<group>.ts` file (skip `confect/_generated/`, `convex/` and `.d.ts`), grep for the four shapes. A Confect function has two parts. The spec (`<group>.spec.ts`) holds the name, `args`, `returns` and `middleware`. The impl (`<group>.impl.ts`) holds the handler. Read both files of the group as one unit.
   (a) identity-from-arg: a public spec (`FunctionSpec.publicQuery(`, `publicMutation(`, `publicAction(`, `publicPaginatedQuery(`, `publicNodeAction(`) whose `args` field map declares `userId`/`actorId`/`ownerId`/`authorId`/`accountId` typed with any `Id(...)` (for example `accountId: Id("accounts")` or `authorId: Id("authors")`), where the impl has no direct auth source. The only auth sources are the `Auth` service (`auth.getUserIdentity`) or the `CurrentUser` service, read inside the matching `FunctionImpl.make` block. A `.middleware(` call is NOT an auth source. Middleware on the function or on its `GroupSpec.make()` chain can be about logging, rate limits or capture, and a group-level `RequireUser` does not stop an impl from using `args.userId`. Regex: `/\b(userId|actorId|ownerId|authorId|accountId)\s*:\s*Id\(/` inside an `args: () => ({ ... })` block, paired with an absent `/\b(Auth|CurrentUser)\b/` in the matching `FunctionImpl.make` block. Also flag any impl that uses `args.<thatName>` for a read or write while it reads neither `CurrentUser` nor `Auth`, even when the spec has middleware. (The word-boundary regex does not match `internalQuery`, `internalMutation` or `internalAction` specs: the `public` prefix excludes them by construction.)
   (b) missing-ownership-check: a public spec whose impl loads a document with `reader.table("<t>").get(args.<xId>)` (an `Id(...)`-typed arg) and then calls `writer.table("<t>").patch`/`.delete`/`.replace` on that same id, or returns the doc's fields directly, with no comparison of any `<doc>.<ownerField>` against the current user (`user._id` from `CurrentUser`, or a value derived from `auth.getUserIdentity`) anywhere in the block (no `===`/`!==` against it).
   (c) PII-leaking public function: a public spec whose `returns` (or the raw doc it returns, such as `<table>.Doc`) includes a sensitive-looking field (`email`, `revenue`, `ssn`, `password`, `token`, `auditLog`, `dashboard`-shaped aggregate) and the function is parameterized by a client-supplied id with no identity check gating access to that id's own scope.
   (d) parent-reference ownership on write: a public mutation spec whose args include an `Id(...)` of a parent/container table (`projectId`, `boardId`, `teamId`, `orgId`, `listId`, `folderId`, `conversationId`, `accountId`, ...) that the impl uses as a foreign key in a `writer.table(...).insert`/`.patch` — attaching or moving a child row into that container — without verifying the caller owns (or is a member of) the referenced parent doc. Creating a row inside someone else's container is the same defect as mutating their row: fixing WHO the caller is (shape a) does not fix WHERE they may write. After handling shapes a-c, re-audit every REMAINING `Id(...)` arg in every public mutation spec for this shape — shape-a fixes routinely leave the parent id arg behind, still unchecked.
   Two more surfaces need the same scan, because middleware does not reach them. First, plain Convex functions (`FunctionSpec.convexPublicQuery`, `convexPublicMutation`, `convexPublicAction`, `convexPublicNodeAction` and the `*Generic` builders in `confect/<group>.ts`): Confect rejects middleware on them, so the auth check must be inside the function. Scan them with the original plain-Convex test: an `args` block with any `v.id(...)` of a user, and no `ctx.auth` reference in the function block. Second, HTTP routes (`confect/http.ts` and the `confect/http/` files): a route is not a spec, so no middleware applies. The handler must read the identity itself. Check that every route that reads or writes user data reads the identity.
   Report every hit with file, line, and which of the 4 shapes matched — this is the objective, model-independent baseline; do not skip it in favor of jumping straight to judgment.
2. HARDEN (foundation-having apps only — see step 0): for each hit, apply the canonical pattern from the Confect patterns file (E5 for the middleware, E3 for the owned-document read) verbatim — do not invent a new helper. Do these moves in order.
   - Add the `RequireUser` middleware if it is absent. Add `confect/middleware/RequireUser.spec.ts` (the `CurrentUser` service, an `Unauthenticated` error, and the `MiddlewareSpec`). Add `confect/middleware/RequireUser.impl.ts` (`MiddlewareImpl.provides`).
   - The impl reads `auth.getUserIdentity`. It maps `NoUserIdentityFoundError` to `Unauthenticated`. It looks the user up with `reader.table("users").get("by_tokenIdentifier", identity.tokenIdentifier)`. This is the Confect form of `requireIdentity(ctx)`.
   - Attach it with `GroupSpec.make().middleware(RequireUser)`. Provide `RequireUserImpl` in the group impl layer. Middleware on a group does not cover subgroups.
   - The E5 `RequireUser` declares `action: false`. A group that has a public action does not typecheck with it attached. For a public action, read the `Auth` service in the action impl and refuse a caller with no identity. Or write the middleware with `MiddlewareImpl.makeByFunctionType` and use `runQuery` in the action branch (E5 rules).
   - Rewrite each flagged function. Delete the client-supplied identity arg. Read `CurrentUser` in the impl instead. The spec change alone is not the fix: the impl must stop using the arg.
   - Add an owned-document read if it is absent, such as `getOwnedNote` in E3. This is the Confect form of `requireOwner(ctx, doc)`. It fails with a typed not-found error if the doc is missing. It fails with the same error if the doc belongs to another user.
   - Choose the owner comparison from the schema. Read the table definition first. If docs store a `users` row id as the owner, resolve the caller's `users` row (the middleware does this) and compare `doc.ownerId` with `user._id`. If docs store the identity string as the owner, compare `doc.ownerId` with `identity.tokenIdentifier` (or with `identity.subject` when the schema stores the subject). Read the identity from the `Auth` service for that case. Never compare an `Id<"users">` field to an identity string. It never matches and silently breaks enforcement. Never compare the identity string to `user._id`.
   - Use `tokenIdentifier` for new keys. Do not key identity on `identity.subject` alone. Report an existing `subject` key as a finding. Do not rewrite a working key.
   - Wrap each `Id(...)`-keyed read or mutate with the owned-document read before touching the row.
   - Scope each PII-returning function through the middleware and an owner check (or an explicit staff/role check) before it reads outside the caller's own scope.
   - For each shape-(d) hit, load the referenced parent doc. Apply the same owner check to it before you insert or patch the child row. When the container models members as an array, use the schema's membership check, for example `participantIds.includes(user._id)`.
   - For a plain Convex function or an HTTP route, put the check inside the function. Read the identity with the raw ctx or the `Auth` service.
   - Confect adds the middleware error schema to every covered ref by itself, on the client side and the server side. Callers add no schema.
   - Never widen scope. An internal/admin function that legitimately operates on an arbitrary user stays an `internal*` spec, never public. Leave it unflagged and unchanged.

   The example below shows the spec and the impl.
   ```ts
   // Bad: the caller chooses whose notes to list.
   FunctionSpec.publicQuery({
     name: "listByUser",
     args: () => ({ userId: Id("users") }),
     returns: () => Schema.Array(notes.Doc),
   });

   // Good (spec): the group has the RequireUser middleware.
   // The function has no identity arg.
   GroupSpec.make()
     .middleware(RequireUser)
     .addFunction(
       FunctionSpec.publicQuery({
         name: "listMine",
         returns: () => Schema.Array(notes.Doc),
       }),
     );

   // Good (impl): the impl reads CurrentUser.
   // The caller cannot name another user.
   const listMine = FunctionImpl.make(databaseSchema, notesSpec, "listMine", () =>
     Effect.gen(function* () {
       const { user } = yield* CurrentUser;
       const reader = yield* DatabaseReader;
       return yield* reader
         .table("notes")
         .index("by_ownerId", (q) => q.eq("ownerId", user._id))
         .collect()
         .pipe(Effect.orDie);
     }),
   );
   ```
3. VERIFY: run `npx confect codegen` and then `npx tsc --noEmit` (or the project's typecheck script) after edits. Codegen success alone is not enough: `GroupImpl.finalize` fails to typecheck until every spec function has an impl and every middleware impl is provided. A hardening pass that doesn't typecheck is not done. Then re-run the step-1 scan to confirm 0 remaining hits (the fixed shapes no longer match the regexes because the middleware or `Auth` now appears and ownership comparisons now exist). Add a negative test for each fixed function with `@confect/test`: a caller with no identity fails with `Unauthenticated`, and a second user cannot read or change the first user's doc (see E16). Seed the other user's data with `confect.run`, and call as the caller with `confect.withIdentity(...)`.
4. Report findings grouped by the 4 rule shapes with file:line, explain why each is exploitable (who could impersonate whom / read whose data), and show the concrete diff applied (or, on a foundationless app, the internalize-and-defer diff plus the auth-setup nudge) — never just describe the fix in prose.

## Rules

- MANDATORY FIRST STEP: before injecting the `RequireUser` middleware or an ownership check, verify the auth foundation exists — a `confect/auth.ts` with a provider AND a users/identities table keyed to the auth identity (`tokenIdentifier` preferred, `subject` accepted and reported as a finding). If either is missing, do not add identity-based enforcement (it's non-functional or mismatched and creates a NEW authz defect); instead convert flagged public admin/privileged specs to internal specs and tell the user to run auth setup first, then re-run this skill.
- Scan objectively before judging — run the 4 deterministic greps first; don't skip straight to LLM judgment, and don't let a clean scan stop you from still eyeballing internal/admin exemptions.
- Identity always comes from the `Auth` service or the `RequireUser` middleware (`CurrentUser`), never from a client-supplied argument — the one legitimate exception is an `internal*` spec that is never exposed publicly.
- Every read or mutate keyed by an `Id(...)` argument must verify ownership server-side (the owned-document read or an inlined equivalent comparison) before touching the row — being logged in is not the same as owning this row.
- Any `Id(...)` argument a public mutation uses as a foreign key when inserting or moving a row must have the referenced parent's ownership (or membership) verified against the caller first — creating a child row inside someone else's project/board/account is the same defect as mutating their row, and it survives an identity-from-arg fix unless checked separately.
- Never leave a public function that returns PII/financial/audit data reachable by an unauthenticated or cross-account client-supplied id.
- Reuse the `RequireUser` middleware and the owned-document read from the Confect patterns file (E5, E3) verbatim — do not fork a parallel helper. Follow the E3 error types: `Unauthenticated` for no identity, and one typed not-found error for both a missing doc and a doc that belongs to another user. This differs from upstream, which used 401, 403 and 404. The fork repo records the reason in `.confect-port/ledger/convex-authz.md`.
- Middleware does not cover subgroups, plain Convex functions or HTTP routes. Check each of them by hand.
- A `.middleware(` call is never an auth source for the scan. Only a direct `Auth` or `CurrentUser` read in the impl clears shape (a).
- `RequireUser` (E5) does not cover public actions. Read `Auth` in the action impl, or use `MiddlewareImpl.makeByFunctionType`.
- Always verify with `confect codegen` and tsc after hardening; a fix that doesn't typecheck is not shipped.
- This is a targeted authz pass, not a general code review — do not expand scope into performance/schema/validator findings; hand those to confect-reviewer.
- SKIP entirely when there is no confect/ directory in the project.
