---
name: confect-auth
description: "Add authentication to a Confect app (Effect on Convex): the confect/auth.ts provider config, the users table, the RequireUser middleware, and client sign-in. Use when the project has a confect/ directory and the user asks for sign-in, login, or an auth provider."
---

<!-- GENERATED from convex-agents content/capabilities/auth.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Add sign-in to the app

Wire sign-in for the current Confect app: a provider, the server config, the client hooks, and a sign-in UI — correctly, including the auth config that's the #1 real-world auth footgun. In Confect, the config is `confect/auth.ts`. Codegen writes `convex/auth.config.ts`. That file re-exports `confect/auth.ts`.

Read `skills/confect/references/confect-patterns.md` (sections E5 and E14) and `skills/confect/references/guidelines.md` first.

## Choose the path

- Path A (verified): an external JWT provider. Write `confect/auth.ts` and read the identity with the `Auth` service. This path needs no library code in `convex/`.
- Path B (not verified in Confect 10.0.0): `@convex-dev/auth` (passkeys, password, OAuth). The library needs a hand-written `convex/auth.ts` module and the `authTables` in `convex/schema.ts`. `npx confect codegen` owns `convex/schema.ts`. It also removes any `convex/*.ts` module that is not a group in the spec. Do not start Path B without the user. Tell the user about both limits and ask which path to take.

## Workflow (Path A)

1. Pick the JWT provider. Get its issuer URL (the `domain`) and the value of the JWT `aud` claim (the `applicationID`).
2. Write `confect/auth.ts`. Do not edit `convex/auth.config.ts`.
3. Run `npx confect codegen`. Set any provider env vars on the deployment. Prefer the Convex MCP `envSet` tool, one call per var. CLI fallback: use the NAME=VALUE form (`npx convex env set "NAME=VALUE"`).
4. Add `confect/tables/users.ts` with a `by_tokenIdentifier` index. Add a `store` mutation that creates the user row for the signed-in identity (E5). Add the `RequireUser` middleware and attach it to each group that needs a signed-in user (E5).
5. Wire the client: `ConvexProviderWithAuth` (not `ConvexProvider`), the sign-in component, and route guards. If you import shadcn/ui primitives (button, input, textarea, label, and so on), add them first with `npx shadcn@latest add <name>`; a missing @/components/ui/* is a hard build error.
6. Verify a sign-in round-trips before declaring done. Call `users.store` signed in. Call a protected function signed out. It must fail with the typed `Unauthenticated` error. Run `npx confect codegen`, then `tsc --noEmit`.

`confect/auth.ts`

```ts
// Codegen writes convex/auth.config.ts. That file imports this file and re-exports it.
export default {
  providers: [
    {
      // The issuer URL of the JWT provider.
      domain: "https://your-auth-provider.example.com",
      // Must equal the JWT aud claim.
      applicationID: "convex",
    },
  ],
};
```

```ts
// Bad: a hand-written convex/auth.config.ts. Codegen overwrites it.
export default { providers: [{ domain: "https://your-auth-provider.example.com", applicationID: "convex" }] };
```

```ts
// Bad: a user id taken from an argument.
args: () => ({ userId: Id("users") })
```

```ts
// Good: read the identity with the Auth service. Key the user by tokenIdentifier.
const auth = yield* Auth;
const identity = yield* auth.getUserIdentity;
```

## Workflow (Path B, only if the user chooses it)

These steps keep the upstream platform rules for `@convex-dev/auth`. Confect does not support them in the verified patterns. Check each step against the codegen limits above.

1. Install @convex-dev/auth (pinned build) and add it to `convex/convex.config.ts`. With pnpm, also `pnpm add jose` (it won't hoist otherwise); you need it for step 3.
2. STOP and tell the user that this step is blocked. Upstream puts the provider in `convex/auth.ts`. Codegen removes that file. No Confect location is verified. Ask the user how to proceed. Do not write the provider anywhere else on your own.
3. Generate the auth keys HEADLESSLY. Do NOT run the interactive `npx @convex-dev/auth` wizard: it needs a login/TTY and hangs in non-interactive, anonymous, or CI runs (the #1 auth time-sink). Generate JWT_PRIVATE_KEY + JWKS deterministically with `jose`:
   node -e 'import("jose").then(async({generateKeyPair,exportPKCS8,exportJWK})=>{const k=await generateKeyPair("RS256",{extractable:true});const priv=await exportPKCS8(k.privateKey);const pub=await exportJWK(k.publicKey);process.stdout.write(JSON.stringify({JWT_PRIVATE_KEY:priv.trimEnd().replace(/\n/g," "),JWKS:JSON.stringify({keys:[{use:"sig",...pub}]})}))})' > .auth-keys.json
   Then set JWT_PRIVATE_KEY and JWKS (from .auth-keys.json) plus SITE_URL on the deployment. Prefer the Convex MCP `envSet` tool, one call per var, to avoid shell-quoting the multi-line key. CLI fallback: use the NAME=VALUE form (`npx convex env set "JWT_PRIVATE_KEY=$JWT"`), NEVER `env set JWT_PRIVATE_KEY "$JWT"` (the value starts with `-----BEGIN` and the CLI parses the leading `-` as an unknown flag). SITE_URL is the dev URL (e.g. http://localhost:3000). Delete .auth-keys.json after.
4. Write `confect/auth.ts` (the silently-always-signed-out bug lives here if it's wrong).
5. Wire the client: ConvexAuthProvider, the sign-in component, and route guards. Add shadcn/ui primitives first, as in Path A step 5.
6. Verify a sign-in round-trips before declaring done.

## Rules

- Write `confect/auth.ts` on every path. A missing or incorrect config makes the app silently always-signed-out. In Confect, `auth.getUserIdentity` always fails with `NoUserIdentityFoundError` without it.
- Never edit `convex/auth.config.ts`. Codegen writes it from `confect/auth.ts`.
- Read the identity with the `Auth` service or with middleware that provides the user. Use `tokenIdentifier` as the stable key. Never take a user id as an argument for authorization.
- Path A: use `ConvexProviderWithAuth` on the client, not `ConvexProvider`. Path B: use `ConvexAuthProvider`.
- Generate JWT_PRIVATE_KEY/JWKS with `jose` (extractable RS256; PKCS8 newlines to spaces; JWKS = {keys:[{use:"sig", ...publicJwk}]}). Do NOT run the interactive `npx @convex-dev/auth` wizard: it hangs headless/anonymous. Set the vars via the MCP `envSet` tool or the NAME=VALUE CLI form. These rules apply to Path B only.
- Path B: passkeys by default; only switch to password/OAuth on explicit request.
- Install any shadcn/ui primitive you import up front (`npx shadcn@latest add ...`); a missing @/components/ui/* is a hard build failure.
- Verify a real sign-in works before finishing.
- Do not start Path B without the user. Codegen removes a hand-written `convex/auth.ts` and owns `convex/schema.ts`.
