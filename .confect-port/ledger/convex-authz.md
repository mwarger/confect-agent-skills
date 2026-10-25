# Ledger: skills/convex-authz/SKILL.md

Source: 36 lines, one file. No references/ directory. Platform-only content: none (all content touches code samples or API names).
Actions: K = keep, U = update, A = add, R = remove.

| Source line(s) | Content | Action | Reason |
| --- | --- | --- | --- |
| 1, 4 | Frontmatter delimiters | K | Unchanged. |
| 2 | name: convex-authz | K | Kept so name matches the directory. The later rename step changes both together. |
| 3 | description | U | Now triggers on Confect work (specs, impls, middleware, codegen). Keeps the 4 shapes and the TRIGGER phrases. SKIP now says no confect/ directory. |
| 6 | GENERATED comment | K | Unchanged, keeps upstream syncs small. |
| 8 | Title | U | "Confect Authz Auditor/Hardener". |
| 10 | Intro, defect statistics | U | Statistics kept word for word. Added a sentence that the measurements come from plain Convex and the shapes are the same in Confect. "requireIdentity/requireOwner from convex-expert.md" replaced by RequireUser middleware and owned-document read (patterns E5, E3). "tsc" now "codegen and tsc". |
| new para after 10 | Confect note | A | Says Confect changes code style, not platform behavior. Points to confect-patterns.md and references/guidelines.md (replaces the Convex guidelines pointer). |
| 12 | "## Workflow" | K | Unchanged. |
| 14 | Step 0, foundation check | U | auth.config.ts becomes confect/auth.ts (copied by codegen, patterns E14). Users table keyed to tokenIdentifier, not subject (patterns rule). "ctx.auth.getUserIdentity() returns null" becomes "auth.getUserIdentity fails with NoUserIdentityFoundError". |
| 14 | Internalize move | U | query/mutation to internalQuery/internalMutation becomes FunctionSpec.public* to internal* (all forms). Added the caller change refs.public to refs.internal and the codegen run, because the API path changes. |
| 14 | "/add auth" nudge | U | Points to the confect-auth skill and patterns E14 and E5. |
| 15 | Step 1 intro | U | Scan targets confect/**/*.spec.ts, *.impl.ts, http files, plain group files. Excludes _generated, convex/, .d.ts. Added the spec plus impl pairing rule, because the handler and args live in different files. |
| 16 | Shape (a) regex | U | `query(`/`mutation(` and `v.id(` become FunctionSpec.public* and `Id(`. "args block" becomes `args: () => ({ ... })`. "ctx.auth absent" becomes no .middleware( on function or group and no Auth service in the impl. Word-boundary note kept in Confect form (public prefix). |
| 17 | Shape (b) | U | ctx.db.get(args.xId) becomes reader.table(t).get(args.xId). patch/delete/replace become writer.table(t) forms. Owner comparison is now against user._id or an Auth-derived value. |
| 18 | Shape (c) | U | `query(`/returns/raw doc becomes public spec/returns/<table>.Doc. ctx.auth check becomes identity check. Sensitive field list kept. |
| 19 | Shape (d) | U | v.id( becomes Id(. ctx.db.insert/patch becomes writer.table(...).insert/.patch. Rest of the rule kept word for word. |
| new, after 19 | Plain functions and HTTP routes | A | Confect middleware does not cover convexPublic* functions or HTTP routes (patterns E5 rules, E12, E13). The old ctx.auth scan is kept for plain functions. |
| 20 | Report every hit | K | Unchanged. |
| 21 | Step 2 | U | Helper file convex/model/auth.ts replaced by RequireUser spec and impl (E5) and owned-document read (E3). requireIdentity maps to the middleware. requireOwner maps to the owned-document read. Error semantics changed: 401/403/404 become typed errors, and missing doc and other-user doc fail the same way (patterns E3). This is the one deliberate semantic change, see open questions. subject becomes tokenIdentifier (patterns rule). Users-row-id note kept. Added: middleware error joins the error union. Added: plain function and HTTP check inside the function. "Never widen scope" kept with internal* specs. |
| new, in step 2 | Bad/Good example | A | AGENTS.md style. Shows the identity-from-arg fix. Sketch, adapted from E5. Not compiled separately. |
| 22 | Step 3 verify | U | `tsc --noEmit` becomes `confect codegen` then tsc. Added the GroupImpl.finalize reason (patterns E2). Re-scan sentence kept. Added negative tests with @confect/test (E16) as the Confect form of convex-test. |
| 23 | Step 4 report | K | Unchanged except no change in meaning. Kept as is. |
| 25 | "## Rules" | K | Unchanged. |
| 27 | Rule: foundation | U | auth.config.ts to confect/auth.ts, subject to tokenIdentifier, internalQuery/internalMutation to internal specs. |
| 28 | Rule: scan first | K | Unchanged. |
| 29 | Rule: identity from ctx.auth | U | ctx.auth becomes Auth service or RequireUser middleware. internal exception kept. |
| 30 | Rule: ownership per _id | U | _id arg becomes Id(...) arg. requireOwner becomes owned-document read. |
| 31 | Rule: parent reference | U | v.id(...) to Id(...). Rest kept. |
| 32 | Rule: no PII leak | U | "public query" to "public function". |
| 33 | Rule: reuse helpers | U | Points to E5 and E3. Added the same-error rule for missing and other-user docs. |
| new | Rule: middleware gaps | A | Middleware does not cover subgroups, plain functions, HTTP routes. |
| 34 | Rule: verify with tsc | U | Adds confect codegen. |
| 35 | Rule: targeted pass | U | convex-reviewer to confect-reviewer (name follows the convex to confect rule). |
| 36 | Rule: SKIP | U | convex/ directory to confect/ directory. |

Removed lines: none removed outright. Raw API forms (ctx.auth, ctx.db, v.id, query/mutation, convex-expert.md helpers) were replaced in place by their Confect forms.

Review fixes (second pass):
- Shape (a): middleware is no longer an auth source. Only a direct Auth or CurrentUser read in the impl clears it. Impls that use the id arg without reading either are flagged. Arg type is any Id(...). Plain-function scan says any v.id(...) and adds convexPublicNodeAction.
- Public actions: RequireUser has action false. Added the fix (Auth in the action impl, or makeByFunctionType) and a rule.
- Owner compare: chosen from the schema (users row id vs identity string). Both cases kept.
- Foundation check: users or identities table, tokenIdentifier or subject, any name. A subject key is a finding, not a missing foundation.
- Removed the invented "callers add the error schema" step. Replaced by a note that Confect adds it.
- Removed references/guidelines.md. The skill now names skills/confect/references/confect-patterns.md and the project's convex/_generated/ai/guidelines.md. Fixed the claim about auth.config.ts (codegen re-exports confect/auth.ts).
- Internalize move lists convexPublic* to convexInternal* and *Generic to internal*Generic.
- Step 2 split into short bullets. Good example now shows the impl.
- Frontmatter name reverted to convex-authz.

Open questions:
1. Error semantics. Upstream used 401, 403 and 404. This skill follows E3: Unauthenticated for no identity, and one not-found error for a missing doc and a foreign doc. Does the team accept this change? It hides doc existence but loses the 403 signal.
2. Other skills still name references/guidelines.md. They are outside this review. Decide whether to fix them in one pass.

Counts: kept 9, updated 23, added 6, removed 0.
