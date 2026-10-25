# Confect guidelines

These guidelines target Confect 10.0.0 (`@confect/core`, `@confect/server`, `@confect/cli`, `@confect/test`) with `effect` 4.0.0 and Convex `^1.44.0`.

Confect is Effect on Convex. Confect changes how you write code. It does not change how the Convex platform behaves. Every Convex limit, index rule, transaction rule, and deploy rule in this file still applies.

- Use this file with `confect-patterns.md` in the same directory. That file has the concept map, the full compiled examples (E1 to E16), and the table of Convex features that Confect does not wrap.
- The Convex guidelines (`npx convex ai-files install`) stay the source for platform rules. When this file and the Convex guidelines disagree on a platform fact, the Convex guidelines win.
- The installed package source (`src/` and `dist/*.d.ts`) is the source for every Confect API name. Do not guess an export.
- Every TypeScript code block in this file was compiled with `tsc --noEmit` (strict, `exactOptionalPropertyTypes`) after `confect codegen`.

## Function guidelines

### Http endpoint syntax

- HTTP endpoints are defined in `confect/http.ts`. Default-export `HttpRouter.make(layer)` from `@confect/server`. Codegen writes `convex/http.ts`. Define routes with `effect/http-api` (`HttpApi`, `HttpApiGroup`, `HttpApiEndpoint`, `HttpApiBuilder`) or `effect/http` (`HttpRouter`). A plain Convex route still uses `httpActionGeneric` from `convex/server`. For example:

`confect/http.ts`

```ts
import { HttpRouter as ConfectHttpRouter } from "@confect/server";
import { httpActionGeneric } from "convex/server";
import * as Layer from "effect/Layer";
import * as HttpRouter from "effect/http/HttpRouter";
import * as HttpServerResponse from "effect/http/HttpServerResponse";

import * as MessagesApi from "./http/MessagesApi";

const http = ConfectHttpRouter.make(
  Layer.mergeAll(
    MessagesApi.layer,
    HttpRouter.add("GET", "/health", HttpServerResponse.text("OK")),
  ),
);

// Escape hatch: a plain Convex route. Exact paths match before the catch-all.
http.route({
  path: "/echo",
  method: "POST",
  handler: httpActionGeneric(async (_ctx, request) => {
    const body = await request.bytes();
    return new Response(body, { status: 200 });
  }),
});

export default http;
```

- In an `HttpApiEndpoint`, the `payload` schema decodes the body. A body that fails decoding gets a 400 response. Do not parse the body by hand. Commit effects through an internal mutation (`refs.internal.*`). See E13 in `confect-patterns.md` for the full `MessagesApi` file.

`confect/http/MessagesApi.ts` (excerpt)

```ts
class MessagesGroup extends HttpApiGroup.make("messages")
  .add(
    HttpApiEndpoint.get("list", "/messages", {
      query: { channel: Schema.String },
      success: Schema.Array(messages.Doc),
    }),
  )
  .add(
    // The payload schema decodes the body. A bad body gets a 400 response.
    HttpApiEndpoint.post("post", "/messages", {
      payload: Schema.Struct({ channel: Schema.String, body: Schema.String }),
      success: Id("messages"),
    }),
  ) {}
```

- In a plain `httpActionGeneric` route, treat the result of `await req.json()` as `unknown` - narrow each field (e.g. `typeof` checks) before use, and return a 400 response for bodies that fail validation.
- `HttpRouter.make` registers one catch-all HTTP action under `/`. The Effect router matches the paths. Endpoint paths are exact. `.prefix("/api")` on the `HttpApi` adds a prefix, so the endpoint `/messages` serves `/api/messages`. A plain `http.route` is always registered at the exact path you specify in the `path` field, and it matches before the catch-all.

### Validators

- In Confect, validators are Effect `Schema` values. Codegen compiles each schema to a Convex validator. Below is an example of an array schema:

`confect/contacts.spec.ts` (excerpt)

```ts
  .addFunction(
    FunctionSpec.publicMutation({
      name: "addValues",
      args: () => ({
        simpleArray: Schema.Array(Schema.Union([Schema.String, Schema.Finite])),
      }),
      returns: () => Schema.Null,
    }),
  )
```

- `Schema.Struct(...)` schemas compose: `.mapFields(Struct.pick([...]))`, `.mapFields(Struct.omit([...]))`, `.mapFields(Struct.map(Schema.optionalKey))` (partial), and `.mapFields(Struct.assign({ d: Schema.String }))` (extend) derive new structs from an existing one. `Struct` comes from `effect/Struct`. Define a shape once and derive variants instead of duplicating fields. Use a struct's `.fields` to supply function `args`. For a table, use `notes.Fields.fields` from `confect/_generated/tables/notes`.

`confect/contacts.spec.ts` (excerpt)

```ts
// Define a shape once. Derive variants with mapFields.
export const Contact = Schema.Struct({
  name: Schema.String,
  email: Schema.String,
  phone: Schema.optionalKey(Schema.String),
});

// pick, omit, and assign (extend) give new structs.
const ContactName = Contact.mapFields(Struct.pick(["name", "email"]));
const ContactWithoutPhone = Contact.mapFields(Struct.omit(["phone"]));
const TaggedContact = Contact.mapFields(Struct.assign({ tag: Schema.String }));
// partial: every field becomes optional.
const ContactChanges = ContactWithoutPhone.mapFields(
  Struct.map(Schema.optionalKey),
);
```

`confect/contacts.spec.ts` (excerpt)

```ts
  .addFunction(
    FunctionSpec.publicMutation({
      name: "rename",
      // Use a struct's fields to supply args.
      args: () => ContactName.fields,
      returns: () => Schema.Null,
    }),
  )
```

- `notes.Doc` (the default export of `confect/_generated/tables/notes`) is the schema for a whole stored document: the table's schema with `_id` and `_creationTime` added, to every member for union tables. Use it when an `args` or `returns` schema needs a complete document instead of re-declaring the fields or the system fields. You do not need `schema.doc` or `docValidator`.
- Below is an example of a table with a schema that codifies a discriminated union type:

`confect/tables/results.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

// A discriminated union table. Each member is a Schema.Struct.
export default Table.make(() =>
  Schema.Union([
    Schema.Struct({
      kind: Schema.Literal("error"),
      errorMessage: Schema.String,
    }),
    Schema.Struct({
      kind: Schema.Literal("success"),
      value: Schema.Finite,
    }),
  ]),
);
```

- Here are the valid Convex types along with their respective Effect schemas:

| Convex Type | TS/JS type | Example Usage | Effect schema for arguments and tables | Notes |
| ----------- | ----------- | -------------------- | --------------------------------------------- | ----- |
| Id | string | `doc._id` | `Id(tableName)` from `confect/_generated/id` | Compiles to `v.id(tableName)`. For system tables, use `GenericId.GenericId("_storage")` from `@confect/core`. |
| Null | null | `null` | `Schema.Null` | JavaScript's `undefined` is not a valid Convex value. Functions the return `undefined` or do not return will return `null` when called from a client. Use `null` instead. Declare `returns: () => Schema.Null`. Codegen rejects `Schema.Undefined` and `Schema.Void`. |
| Int64 | bigint | `3n` | `Schema.BigInt` | Int64s only support BigInts between -2^63 and 2^63-1. Convex supports `bigint`s in most modern browsers. |
| Float64 | number | `3.1` | `Schema.Finite` | Convex supports all IEEE-754 double-precision floating point numbers (such as NaNs). Inf and NaN are JSON serialized as strings. `Schema.Finite` rejects NaN and Infinity. `Schema.Number` also compiles to `v.float64()` and accepts them. Use `Schema.Number` only when you must store them. |
| Boolean | boolean | `true` | `Schema.Boolean` | |
| String | string | `"abc"` | `Schema.String` | Strings are stored as UTF-8 and must be valid Unicode sequences. Strings must be smaller than the 1MB total size limit when encoded as UTF-8. Checks such as `.check(Schema.isMaxLength(2000))` run in Confect, not in the Convex validator. |
| Bytes | ArrayBuffer | `new ArrayBuffer(8)` | `Schema.instanceOf(ArrayBuffer)` | Convex supports first class bytestrings, passed in as `ArrayBuffer`s. Bytestrings must be smaller than the 1MB total size limit for Convex types. |
| Array | Array | `[1, 3.2, "abc"]` | `Schema.Array(values)` | Arrays can have at most 8192 values. |
| Object | Object | `{a: "abc"}` | `Schema.Struct({property: value})` | Convex only supports "plain old JavaScript objects" (objects that do not have a custom prototype). Objects can have at most 1024 entries. Field names must be nonempty and not start with "$" or "\_". |
| Record | Record | `{"a": "1", "b": "2"}` | `Schema.Record(keys, values)` | Records are objects at runtime, but can have dynamic keys. Keys must be only ASCII characters, nonempty, and not start with "$" or "\_". Use a string key schema. In 10.0.0, `Id(tableName)` is not a valid key schema. |

- For optional fields, nullable fields, literals, unions, and transformations (`URL`, `Option`, `DateTime`), see the validators table in `confect-patterns.md`.

### Function registration

- Use `FunctionSpec.internalQuery`, `FunctionSpec.internalMutation`, and `FunctionSpec.internalAction` from `@confect/core` to declare internal functions in a group spec (`confect/<group>.spec.ts`). These functions are private and aren't part of an app's API. They can only be called by other Convex functions. Implement each one with `FunctionImpl.make` from `@confect/server` in `confect/<group>.impl.ts`. `confect codegen` registers them in `convex/`.
- Use `FunctionSpec.publicQuery`, `FunctionSpec.publicMutation`, and `FunctionSpec.publicAction` to declare public functions. These functions are part of the public API and are exposed to the public Internet. Do NOT use the `public*` constructors to declare sensitive internal functions that should be kept private. A function invoked only by your own code - e.g. the mutation an HTTP route calls to commit its effects - is internal, not public.
- You CANNOT register a function through the `refs` object. Codegen registers every function that a spec declares. `GroupImpl.finalize` does not typecheck until every spec function has an impl.
- Confect validates the arguments of every Confect function. Declare `args: () => ({ ... })` as a field map, not as a `Schema.Struct`. An omitted `args` means "no arguments", and Confect still validates that. Every function also declares `returns`.
- ALWAYS include argument validators in plain Convex functions (`confect/<group>.ts`, declared with the `FunctionSpec.convex*` constructors). Confect does not decode or encode their arguments. See E12 in `confect-patterns.md`.

### Function calling

- Use `runQuery` from the `QueryRunner` service to call a query from a query, mutation, action, or HTTP action.
- Use `runMutation` from the `MutationRunner` service to call a mutation from a mutation, action, or HTTP action.
- Use `runAction` from the `ActionRunner` service to call an action from an action or HTTP action.
- ONLY call an action from another action if you need to cross runtimes (e.g. from V8 to Node). Otherwise, pull out the shared code into an Effect helper and call that directly instead.
- Try to use as few calls from actions to queries and mutations as possible. Queries and mutations are transactions, so splitting logic up into multiple calls introduces the risk of race conditions.
- All of these calls take a ref from `confect/_generated/refs` (`refs.public.*` or `refs.internal.*`). Do NOT try to pass the impl or the callee function directly into one of these calls.
- Nested `runQuery` and `runMutation` calls from a mutation execute as subtransactions. If a nested call fails, its writes roll back independently, so the caller can catch the error and continue with its own writes intact. A declared error of the nested function comes back decoded; catch it with `Effect.catchTag`. Any other failure comes back as a defect; catch it with `Effect.catchDefect`.
- In Convex 1.41+, `runQuery` and `runMutation` accept an optional third argument with `transactionLimits`. These limits cap how much the nested call may additionally consume on top of what the caller has already used - they can only tighten the global transaction limits, never raise them. If the nested call exceeds its cap and rolls back, the caller keeps its own remaining budget, which is useful for preserving caller headroom. For example:

`confect/messages.impl.ts` (excerpt)

```ts
const postBounded = FunctionImpl.make(
  databaseSchema,
  messages,
  "postBounded",
  ({ channel, body }) =>
    Effect.gen(function* () {
      const { runMutation } = yield* MutationRunner;

      return yield* runMutation(
        refs.internal.messages.postSystem,
        { channel, body },
        { transactionLimits: { documentsWritten: 1 } },
      ).pipe(Effect.catchDefect(() => Effect.succeed(null)));
    }).pipe(Effect.orDie),
);
```

The supported `transactionLimits` fields are `bytesRead`, `bytesWritten`, `databaseQueries`, `documentsRead`, `documentsWritten`, `functionsScheduled`, and `scheduledFunctionArgsBytes`.

- Refs carry the args, returns, and error schemas. You do not need a return type annotation when you call a function in the same group.

### Function references

- Use `refs.public` (the default export of `confect/_generated/refs`) to call public functions declared with `FunctionSpec.publicQuery`, `publicMutation`, or `publicAction`.
- Use `refs.internal` to call internal (or private) functions declared with `FunctionSpec.internalQuery`, `internalMutation`, or `internalAction`.
- Confect uses file-based routing from the spec files, so a public function named `f` in `confect/example.spec.ts` has the ref `refs.public.example.f` and the Convex function name `example:f`.
- A private function named `g` in `confect/example.spec.ts` has the ref `refs.internal.example.g`.
- Functions can also be declared in directories nested within the `confect/` folder. For example, a public function `h` declared in `confect/messages/access.spec.ts` has the ref `refs.public.messages.access.h`.
- The file path is the API path. A rename of a spec file changes the API.

### Pagination

- Declare a paginated query with `FunctionSpec.publicPaginatedQuery` (or `internalPaginatedQuery`) and an `item` schema. Confect adds the `paginationOpts` argument and the result schema:

`confect/notes.spec.ts` (excerpt)

```ts
  .addFunction(
    FunctionSpec.publicPaginatedQuery({
      name: "listMineByTag",
      args: () => ({ tag: Schema.String }),
      item: () => notes.Doc,
    }),
  )
```

`confect/notes.impl.ts` (excerpt)

```ts
const listMineByTag = FunctionImpl.make(
  databaseSchema,
  notes,
  "listMineByTag",
  ({ tag, paginationOpts }) =>
    Effect.gen(function* () {
      const { user } = yield* CurrentUser;
      const reader = yield* DatabaseReader;

      // Pass paginationOpts unchanged.
      return yield* reader
        .table("notes")
        .index(
          "by_authorId_and_tag",
          (q) => q.eq("authorId", user._id).eq("tag", tag),
          "desc",
        )
        .paginate(paginationOpts);
    }).pipe(Effect.orDie),
);
```

Note: `paginationOpts` is an object with the following properties:

- `numItems`: the initial page-size target — not a guaranteed maximum under reactive pagination (the schema is `Schema.Finite`)
- `cursor`: the cursor to use to fetch the next page of documents; required (the schema is `Schema.Union([Schema.String, Schema.Null])`)
- `endCursor` (optional): bounds the page to end at a known cursor
- `maximumRowsRead` (optional): limits how many rows the query may scan before returning a partial page
- `maximumBytesRead` (optional): limits how many bytes the query may read before returning a partial page
- `id` (optional): client-managed pagination metadata accepted by `PaginationOptions.PaginationOptions`

Do not declare `paginationOpts` in `args`. Confect adds it with the `PaginationOptions.PaginationOptions` schema from `@confect/core`, and a redeclared field fails to typecheck. Pass `paginationOpts` unchanged to `.paginate()` — do not reconstruct it field by field, or the optional fields lose their native behavior.

```ts
// Bad: redeclares paginationOpts. This fails to typecheck.
args: () => ({ tag: Schema.String, paginationOpts: PaginationOptions.PaginationOptions }),

// Good: declare only your own args. Confect adds paginationOpts.
args: () => ({ tag: Schema.String }),
```

A query that ends in `.paginate()` returns an object that has the following properties:

- `page`: an array of the documents fetched for this page
- `isDone`: a boolean representing whether this is the last page of documents
- `continueCursor`: a string cursor to fetch the next page of documents
- `splitCursor` (optional, string or null) and `pageStatus` (optional, `"SplitRecommended"`, `"SplitRequired"`, or null): present when the page was cut short and should be split

Confect builds the return schema of a paginated query with `PaginationResult.PaginationResult(item)` from `@confect/core`. Do not reproduce this shape by hand.

## Schema guidelines

- Always define each table in its own file in `confect/tables/<name>.ts`. The file name is the table name. Default-export a `Table.make(...)`. `confect codegen` writes `convex/schema.ts`. Do not edit it.
- Always import `Table` from `@confect/core` and `Schema` from `effect/Schema`. For example:

`confect/tables/notes.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

import { Id } from "../_generated/id";

export default Table.make(() =>
  Schema.Struct({
    authorId: Id("users"),
    text: Schema.String.check(Schema.isMaxLength(2000)),
    tag: Schema.optionalKey(Schema.String),
  }),
)
  .index("by_authorId", ["authorId"])
  .index("by_authorId_and_tag", ["authorId", "tag"]);
```

- System fields are automatically added to all documents and are prefixed with an underscore. The two system fields that are automatically added to all documents are `_creationTime` which has the schema `Schema.Finite` and `_id` which has the schema `Id(tableName)`.
- Always include all index fields in the index name. For example, if an index is defined as `["field1", "field2"]`, the index name should be "by_field1_and_field2".
- Index fields must be queried in the same order they are defined. If you want to be able to query by "field1" then "field2" and by "field2" then "field1", you must create separate indexes.
- Do not store unbounded lists as an array field inside a document (e.g. `Schema.Array(Schema.Struct({...}))`). As the array grows it will hit the 1MB document size limit, and every update rewrites the entire document. Instead, create a separate table for the child items with a foreign key back to the parent.
- Separate high-churn operational data (e.g. heartbeats, online status, typing indicators) from stable profile data. Storing frequently updated fields on a shared document forces every write to contend with reads of the entire document. Instead, create a dedicated table for the high-churn data with a foreign key back to the parent record.

- Adding an index to a large existing table blocks the deploy until backfill completes. In plain Convex, a staged index (`staged: true`) backfills asynchronously without blocking. Confect 10.0.0 cannot declare a staged index: `Table.index(name, fields)` has no options argument, and codegen owns `convex/schema.ts`. Plan for the deploy to block while a new index backfills on a large table.

## Authentication guidelines

- Convex supports JWT-based authentication through `convex/auth.config.ts`. In Confect, write this config as `confect/auth.ts`. Codegen writes `convex/auth.config.ts` as a re-export of this file. Do not edit `convex/auth.config.ts`. Codegen overwrites it on every run. ALWAYS create this file when using authentication. Without it, `auth.getUserIdentity` always fails with `NoUserIdentityFoundError`.
- Example `confect/auth.ts`:

`confect/auth.ts`

```ts
// Codegen writes convex/auth.config.ts as a re-export of this file.
export default {
  providers: [
    {
      domain: "https://your-auth-provider.example.com",
      applicationID: "convex",
    },
  ],
};
```

The `domain` must be the issuer URL of the JWT provider. Convex fetches `{domain}/.well-known/openid-configuration` to discover the JWKS endpoint. The `applicationID` is checked against the JWT `aud` (audience) claim.

- Use the `Auth` service from `confect/_generated/services` to get the authenticated user's identity in any query, mutation, action, or HTTP action. `auth.getUserIdentity` is an Effect, not a function. It fails with `NoUserIdentityFoundError` if the user is not authenticated, or succeeds with a `UserIdentity` object with fields like `subject`, `issuer`, `name`, `email`, etc. The `subject` field is the unique user identifier.

`confect/users.impl.ts` (excerpt)

```ts
// Create the user row for the signed-in identity, or return the existing one.
const store = FunctionImpl.make(databaseSchema, users, "store", () =>
  Effect.gen(function* () {
    const auth = yield* Auth;
    const reader = yield* DatabaseReader;
    const writer = yield* DatabaseWriter;

    const identity = yield* auth.getUserIdentity.pipe(
      Effect.catchTag("NoUserIdentityFoundError", () =>
        Effect.fail(new NotSignedIn()),
      ),
    );

    const existing = yield* reader
      .table("users")
      .index("by_tokenIdentifier", (q) =>
        q.eq("tokenIdentifier", identity.tokenIdentifier),
      )
      .first()
      .pipe(Effect.orDie);

    if (Option.isSome(existing)) {
      return existing.value._id;
    }

    return yield* writer
      .table("users")
      .insert({
        tokenIdentifier: identity.tokenIdentifier,
        name: identity.name ?? "Anonymous",
        ...(identity.email === undefined ? {} : { email: identity.email }),
      })
      .pipe(Effect.orDie);
  }),
);
```

- In Convex `UserIdentity`, `tokenIdentifier` is guaranteed and is the canonical stable identifier for the authenticated identity. For any auth-linked database lookup or ownership check, prefer `identity.tokenIdentifier` over `identity.subject`. Do NOT use `identity.subject` alone as a global identity key.
- NEVER accept a `userId` or any user identifier as a function argument for authorization purposes. Always derive the user identity server-side through the `Auth` service, or through a middleware that provides the current user (see E5 in `confect-patterns.md`).
- Confect does not change client auth setup. When using an external auth provider with Convex on the client, use `ConvexProviderWithAuth` instead of `ConvexProvider`:

```tsx
import { ConvexProviderWithAuth, ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(process.env.NEXT_PUBLIC_CONVEX_URL!);

function App({ children }: { children: React.ReactNode }) {
  return (
    <ConvexProviderWithAuth client={convex} useAuth={useYourAuthHook}>
      {children}
    </ConvexProviderWithAuth>
  );
}
```

The `useAuth` prop must return `{ isLoading, isAuthenticated, fetchAccessToken }`. Do NOT use plain `ConvexProvider` when authentication is needed — it will not send tokens with requests.

## Typescript guidelines

- Use the type `GenericId.GenericId<"users">` from `@confect/core` to get the type of the id for a given table. Use the schema `Id("users")` from `confect/_generated/id` in args, returns, and tables. In 10.0.0, `confect/_generated/id` exports no `Id` type.
- Use `UsersDoc` from `confect/_generated/docs` (or `typeof users.Doc.Type`) to get the full document type for a table.
- Confect handlers do not take a `ctx` parameter. Get each capability as a service from `confect/_generated/services` (`DatabaseReader`, `DatabaseWriter`, `Auth`, `QueryRunner`, `Scheduler`, `Storage`, and so on). Use the raw ctx services `QueryCtx`, `MutationCtx`, and `ActionCtx` only for escape hatches, such as component clients and `db.normalizeId`. NEVER use `any` for a ctx — always use the proper service.

```ts
// Bad: raw ctx for a plain read.
const ctx = yield* QueryCtx;
const note = yield* Effect.promise(() => ctx.db.get(noteId));

// Good: use the DatabaseReader service.
const reader = yield* DatabaseReader;
const note = yield* reader.table("notes").get(noteId);
```
- If you need to define a `Record` make sure that you correctly provide the type of the key and value in the type. In 10.0.0, `Schema.Record` does not accept `Id("users")` as a key schema. Use `Schema.Record(Schema.String, Schema.String)`, which compiles to `v.record(v.string(), v.string())`, and type the value in code as `Record<GenericId.GenericId<"users">, string>`. ```ts
// Bad: Id(...) is not a valid Record key schema in 10.0.0.
Schema.Record(Id("users"), Schema.String)

// Good: use a string key. Type the key as an id in code.
Schema.Record(Schema.String, Schema.String)
```

Below is an example of using `Record` with an id type in a query:

`confect/directory.spec.ts` (excerpt)

```ts
  .addFunction(
    FunctionSpec.internalQuery({
      name: "namesById",
      args: () => ({ userIds: Schema.Array(Id("users")) }),
      // Id("users") is not a valid record key in 10.0.0. Use a string key.
      returns: () => Schema.Record(Schema.String, Schema.String),
    }),
  )
```

`confect/directory.impl.ts` (excerpt)

```ts
const namesById = FunctionImpl.make(
  databaseSchema,
  directory,
  "namesById",
  ({ userIds }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;
      const idToName: Record<GenericId.GenericId<"users">, string> = {};

      for (const userId of userIds) {
        const user = yield* reader.table("users").get(userId).pipe(Effect.option);
        if (Option.isSome(user)) {
          idToName[user.value._id] = user.value.name;
        }
      }

      return idToName;
    }),
);
```

- Be strict with types, particularly around id's of documents. For example, if a function takes in an id for a document in the 'users' table, take in `Id("users")` rather than `Schema.String`, and type it as `GenericId.GenericId<"users">` rather than `string`.
- Confect does not surface typed app environment variables (`defineApp({ env })` with `env` from `convex/_generated/server`). Read environment variables with `Config` from `effect/Config` instead of `process.env`. Confect installs a `ConfigProvider` for the Convex runtime. Use `Config.schema` when you need a decoded type. The platform-provided `CONVEX_SITE_URL` and `CONVEX_CLOUD_URL` are always set; never declare them in `convex.config.ts` (redeclaring them fails the deploy). You can still declare other variables with `defineApp({ env })` in `convex/convex.config.ts`. You own that file, and codegen does not overwrite it. Read those variables with `Config`. Node actions use Effect's default `ConfigProvider`.

`confect/settings.impl.ts` (excerpt)

```ts
// Confect installs a Convex-aware ConfigProvider. Read env vars with Config.
const describe = FunctionImpl.make(databaseSchema, settings, "describe", () =>
  Effect.gen(function* () {
    const siteUrl = yield* Config.String("CONVEX_SITE_URL");
    const maxResults = yield* Config.Int("MAX_RESULTS").pipe(
      Config.withDefault(20),
    );
    return { siteUrl, maxResults };
  }).pipe(Effect.orDie),
);
```

## Full text search guidelines

- A query for "10 messages in channel '#general' that best match the query 'hello hi' in their body" would look like this, with `channel` and `query` passed as arguments:

`confect/messages.impl.ts` (excerpt)

```ts
const search = FunctionImpl.make(
  databaseSchema,
  messages,
  "search",
  ({ channel, query }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;

      return yield* reader
        .table("messages")
        .search("search_body", (q) =>
          q.search("body", query).eq("channel", channel),
        )
        .take(10);
    }).pipe(Effect.orDie),
);
```

## Vector search guidelines

- Store embeddings in a field with the schema `Schema.Array(Schema.Finite)` (it compiles to `v.array(v.float64())`) and declare a vector index on it in the table file:

`confect/tables/documents.ts`

```ts
import { Table } from "@confect/core";
import * as Schema from "effect/Schema";

export default Table.make(() =>
  Schema.Struct({
    title: Schema.String,
    category: Schema.String,
    embedding: Schema.Array(Schema.Finite),
  }),
).vectorIndex("by_embedding", {
  vectorField: "embedding",
  dimensions: 1536,
  filterFields: ["category"],
});
```

- `dimensions` must exactly match the length of the vectors you store and search with.
- The `VectorSearch` service is ONLY available in actions - not in queries or mutations:

`confect/documents.impl.ts` (excerpt)

```ts
const similar = FunctionImpl.make(
  databaseSchema,
  documents,
  "similar",
  ({ embedding, category }) =>
    Effect.gen(function* () {
      const vectorSearch = yield* VectorSearch;
      const { runQuery } = yield* QueryRunner;

      const hits = yield* vectorSearch("documents", "by_embedding", {
        vector: [...embedding],
        limit: 10,
        filter: (q) => q.eq("category", category),
      });

      const docs = yield* runQuery(refs.internal.documents.getMany, {
        ids: hits.map((hit) => hit._id),
      });
      const byId = new Map(docs.map((doc) => [doc._id, doc]));

      return hits.flatMap((hit) => {
        const document = byId.get(hit._id);
        return document === undefined ? [] : [{ document, score: hit._score }];
      });
    }).pipe(Effect.orDie),
);
```

- The vector search `filter` supports only equality on declared `filterFields` and `q.or(...)` - there is no AND across different fields and no inequality. Push what you can into the vector filter and apply any remaining predicates after hydration.
- Vector search returns only `{ _id, _score }` pairs ordered by descending similarity score - not full documents. Because actions have no `DatabaseReader`, hydrate the hits through an internal query, preserve the vector search's order, and pair each score with its document by ID.

## Component guidelines

- Convex components are installable building blocks (e.g. `@convex-dev/aggregate`, `@convex-dev/rate-limiter`) with their own isolated tables and functions. Install the npm package, then mount the component in `convex/convex.config.ts`. You own this file, and Confect codegen does not overwrite it:

```ts
import { defineApp } from "convex/server";
import aggregate from "@convex-dev/aggregate/convex.config"; // no .js suffix

const app = defineApp();
app.use(aggregate);
export default app;
```

- After mounting, the `components` registry in `confect/_generated/components` references the component (e.g. `components.aggregate`), and is passed to the component's client class. Do not import `components` from `convex/_generated/api`.

```ts
// Bad: Confect code does not use the Convex generated registry.
import { components } from "../convex/_generated/api";

// Good: use the Confect components registry.
import { components } from "./_generated/components";
```

`confect/rateLimits.ts`

```ts
import { MINUTE, RateLimiter } from "@convex-dev/rate-limiter";

import { components } from "./_generated/components";

// Use the Confect components registry, not convex/_generated/api.
export const rateLimiter = new RateLimiter(components.rateLimiter, {
  sendMessage: { kind: "token bucket", rate: 10, period: MINUTE, capacity: 3 },
});
```

- Component functions are not exposed to clients; the app's own queries and mutations wrap them. Perform authentication and authorization in the app functions before calling into a component.
- Component reads and writes participate in the calling mutation's transaction. When a component mirrors state from one of your tables (like an aggregate over a table), update the component in the SAME mutation as every insert, patch, replace, or delete of that table - never from a separate function - so the two can never drift. Component clients take the raw Convex ctx: get it with `yield* MutationCtx` (or `QueryCtx`, `ActionCtx`) and wrap the call in `Effect.promise`:

`confect/messages.impl.ts` (excerpt)

```ts
const send = FunctionImpl.make(
  databaseSchema,
  messages,
  "send",
  ({ channel, author, body }) =>
    Effect.gen(function* () {
      // Component clients take the raw Convex ctx. The component write is part
      // of this mutation's transaction.
      const ctx = yield* MutationCtx;
      const status = yield* Effect.promise(() =>
        rateLimiter.limit(ctx, "sendMessage", { key: author }),
      );
      if (!status.ok) {
        return yield* new RateLimited({ retryAfterMs: status.retryAfter });
      }

      const writer = yield* DatabaseWriter;
      return yield* writer
        .table("messages")
        .insert({ channel, author, body })
        .pipe(Effect.orDie);
    }),
);
```

- To author a LOCAL component: a directory with its own `convex.config.ts` (`export default defineComponent("myName");` - the argument is the name string), its own `schema.ts`, and functions built from that directory's own `_generated/server`. Mount it from the root config (`app.use(myName)` - no options), and reference its functions through the generated `components` object INCLUDING the module segment: a function in `myName/index.ts` is `components.myName.index.myFunction`, never `components.myName.myFunction`. Confect codegen does not build component code, so write the component in plain Convex. Codegen also removes every `.ts` file under `convex/` that is not a Confect group module (it keeps only `schema.ts`, `http.ts`, `crons.ts`, `auth.config.ts`, `convex.config.ts`, and files in `_generated/` directories). Do not put a local component's function files under `convex/`. Keep the component in its own directory outside `convex/`, and import its `convex.config.ts` from `convex/convex.config.ts`. Call it from Confect through the raw ctx.
- For per-key quotas, cooldowns, or throttling (N operations per period, retry-after), use the `@convex-dev/rate-limiter` component - hand-rolled counter or window-scan implementations admit races under concurrency and lose quota when a mutation fails.
- For chat or assistant features where an LLM replies inside a durable conversation - per-user resumable histories, recorded tool-call steps, several assistants sharing one conversation - use the `@convex-dev/agent` component: mount it, create one component thread per conversation, and generate/read through it (`createThread(ctx, components.agent, ...)`, `new Agent(components.agent, { name, languageModel, tools }).generateText(ctx, { threadId }, { prompt })`, `listMessages`). Do not hand-roll a messages table or call an LLM SDK directly from your functions for these. In Confect, `ctx` is the raw ctx service (`yield* ActionCtx` or `yield* MutationCtx`), and `components` comes from `confect/_generated/components`.
- For async Convex functions needing bounded parallelism, serialized mutation work, or completion callbacks, use `@convex-dev/workpool`; retry only idempotent actions.
- For ephemeral presence - who is online/viewing/typing in a room, tracked by client heartbeats with session tokens, multi-session aggregation (one entry per user across tabs), and timeout-to-offline - use the `@convex-dev/presence` component - hand-rolled lastSeen tables need wall-clock query filters that go stale, and per-session rows break the one-entry-per-user contract.
- Calling a component mutation is a subtransaction: if it throws and the caller catches the error, the component's writes roll back while the calling mutation continues and can still commit its own writes. In Confect, `Effect.promise` turns the thrown error into a defect. Catch it with `Effect.catchDefect`.
- To pass a function across a component boundary, mint a handle in the app. Confect does not wrap `createFunctionHandle`. Call it from `convex/server` with the ref's function reference: `createFunctionHandle(Ref.getFunctionReference(refs.internal.index.myCallback))` (`Ref` from `@confect/core`; async, takes only the function reference - `getFunctionHandle` and `getFunctionName` are not this API). Send it as a string; the receiver casts it back and invokes it with the raw ctx: `await ctx.runMutation(args.handle as FunctionHandle<"mutation">, callbackArgs);`.

`confect/maintenance.impl.ts` (excerpt)

```ts
// Escape hatch: mint a function handle to pass across a component boundary.
const clearResultsHandle = FunctionImpl.make(
  databaseSchema,
  maintenance,
  "clearResultsHandle",
  () =>
    Effect.promise(() =>
      createFunctionHandle(
        Ref.getFunctionReference(refs.internal.maintenance.clearResults),
      ),
    ),
);
```

## Query guidelines

- Every Confect query names an index: `reader.table(t).index(name, (q) => ..., order?)`. Express every predicate supported by the index in its index range. Confect queries have no `.filter()`. For additional predicates that cannot be expressed by that index, use `.stream()` with `Stream.filter` and `Stream.take`, or the `filter` argument of `.paginate(opts, filter)`. Filtering happens after the index scan and does not reduce rows read, so it does not make an otherwise unbounded query scalable.
- Do not read the wall clock inside a query. Queries are not rerun merely because time advances, so results derived from `Date.now()` or a zero-argument `new Date()` can become stale, and wall-clock reads also reduce query-cache reuse. Instead, pass the current time in as an argument and let the client refresh it, or materialize time-based state with scheduled mutations that update a flag field. (`Date.now()` is fine in mutations and actions.) In Confect 10.0.0, `Date.now()` and `Clock.currentTimeMillis` in a query read the real time; they are not stubbed. `Effect.timed` in a query reports zero elapsed time. See E7 in `confect-patterns.md`.
- If the user does not explicitly tell you to return all results from a query you should ALWAYS return a bounded collection instead. So that is instead of using `.collect()` you should use `.take()` or paginate on database queries. This prevents future performance issues when tables grow in an unbounded way.
- Never use `.collect().length` to count rows. Convex has no built-in count operator. For a simple total, maintain a denormalized counter document updated in your mutations. When queries need aggregates over many rows - counts, sums, ranks/positions, or offset access, whole-table or within a key range - use the `@convex-dev/aggregate` component (O(log n) reads; keep it updated in the same mutation as every source-table write).
- Convex queries do NOT support `.delete()`. To delete all documents matching a query, read them (in `.take(n)` batches or with `.stream()`) and call `writer.table("tasks").delete(row._id)` on each.
- Convex mutations are transactions with limits on the documents and bytes they read and write. If a mutation needs to process more documents than fit in a single transaction (e.g. bulk deletion on a large table), process one batch, then `yield* scheduler.runAfter(Duration.zero, refs.internal.myModule.myMutation, args)` to continue in a fresh transaction. A fixed `.take(n)` batch is the default when document sizes are uniform (see `purgeCompleted` in E7 of `confect-patterns.md`); when they vary, iterate with `.stream()` and `Stream.runForEachWhile`, and after each write `yield* transaction.getMetrics()` (from the `TransactionMetadata` service), scheduling the continuation and stopping as soon as any needed `.remaining` metric (e.g. `metrics.bytesRead.remaining`) falls to a safety reserve:

`confect/tasks.impl.ts` (excerpt)

```ts
const BYTES_READ_RESERVE = 512 * 1024;

// Documents vary in size: stream them, check the metrics after each write,
// and schedule a continuation before the read limit.
const purgeCompletedBySize = FunctionImpl.make(
  databaseSchema,
  tasks,
  "purgeCompletedBySize",
  () =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;
      const writer = yield* DatabaseWriter;
      const transaction = yield* TransactionMetadata;
      const scheduler = yield* Scheduler;

      const completed = reader
        .table("tasks")
        .index("by_isCompleted", (q) => q.eq("isCompleted", true))
        .stream();

      yield* Stream.runForEachWhile(completed, (task) =>
        Effect.gen(function* () {
          yield* writer.table("tasks").delete(task._id);

          const metrics = yield* transaction.getMetrics();
          if (metrics.bytesRead.remaining > BYTES_READ_RESERVE) {
            return true;
          }

          yield* scheduler.runAfter(
            Duration.zero,
            refs.internal.tasks.purgeCompletedBySize,
          );
          return false;
        }),
      );

      return null;
    }).pipe(Effect.orDie),
);
```

- Use `reader.table(t).get(indexName, ...values)` to get a single document by an index. It replaces `.unique()`. It fails with `GetByIndexFailure` when no document matches. More than one match is a defect.

`confect/directory.impl.ts` (excerpt)

```ts
// get(indexName, value) replaces .unique(). No match fails with
// GetByIndexFailure. More than one match is a defect.
const findByToken = FunctionImpl.make(
  databaseSchema,
  directory,
  "findByToken",
  ({ tokenIdentifier }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;
      return yield* reader
        .table("users")
        .get("by_tokenIdentifier", tokenIdentifier);
    }).pipe(
      Effect.catchTag("GetByIndexFailure", () => Effect.succeed(null)),
      Effect.orDie,
    ),
);
```

- Confect has no `for await` over a query. When you iterate with `.stream()`, don't use `.collect()` or `.take(n)` on the query. Process the `Stream` with Stream operators (for example `Stream.runForEachWhile`) instead.

### Ordering

- Queries default to ascending order over the selected index key. Confect has no plain table scan: use the built-in `by_creation_time` index (`.index("by_creation_time")`), which returns documents in ascending `_creationTime` order; a query using a custom index defaults to ascending order across that index's entire key.
- You can pass `"asc"` or `"desc"` as the last argument of `.index(...)` to pick whether a query is in ascending or descending order. If the order isn't specified, it defaults to ascending.
- Document queries that use indexes will be ordered based on the columns in the index and can avoid slow table scans.
- Convex appends `_creationTime` as the final column of every database index. An index on `["points"]` therefore orders by `points`, then `_creationTime`. A `"desc"` order reverses the entire index key, so rows with equal `points` come back newest first. Rely on this built-in tiebreak instead of re-sorting results in JavaScript.

## Mutation guidelines

- Use `writer.table("tasks").replace(taskId, {...})` from the `DatabaseWriter` service to fully replace an existing document. If the document does not exist, Convex throws, and Confect turns the error into a defect. Syntax:

`confect/tasks.impl.ts` (excerpt)

```ts
// replace swaps the whole document. A missing document is a defect.
const reset = FunctionImpl.make(
  databaseSchema,
  tasks,
  "reset",
  ({ taskId, dueAt }) =>
    Effect.gen(function* () {
      const writer = yield* DatabaseWriter;

      yield* writer
        .table("tasks")
        .replace(taskId, { text: "Buy milk", isCompleted: false, dueAt });

      return null;
    }).pipe(Effect.orDie),
);
```

- Use `writer.table("tasks").patch(taskId, { isCompleted: true })` to shallow merge updates into an existing document. It fails with `GetByIdFailure` if the document does not exist. An optional field set to `undefined` is removed. Confect's `patch` reads and decodes the document, then writes the merged document, so a patch also counts as a read. Syntax:

`confect/tasks.impl.ts` (excerpt)

```ts
// patch merges fields. It fails with GetByIdFailure when the document does
// not exist.
const markCompleted = FunctionImpl.make(
  databaseSchema,
  tasks,
  "markCompleted",
  ({ taskId }) =>
    Effect.gen(function* () {
      const writer = yield* DatabaseWriter;

      yield* writer
        .table("tasks")
        .patch(taskId, { isCompleted: true })
        .pipe(
          Effect.catchTags({
            GetByIdFailure: () => Effect.fail(new TaskNotFound({ taskId })),
            DocumentDecodeError: Effect.die,
            DocumentEncodeError: Effect.die,
          }),
        );

      return null;
    }),
);
```

## Action guidelines

- Always put actions that use Node.js built-in modules in a Node group: `GroupSpec.makeNode()` with `FunctionSpec.publicNodeAction` or `FunctionSpec.internalNodeAction`. Codegen adds `"use node";` to the generated module. See E11 in `confect-patterns.md`.
- A Node group holds only Node actions; its `addFunction` rejects other functions at the type level. Only actions can run in the Node.js runtime; queries and mutations must stay in the default Convex runtime. If you need Node.js built-ins alongside queries or mutations, put the action in a separate Node group.
- `fetch()` is available in the default Convex runtime. You do NOT need a Node group just to use `fetch()`.
- Never use `DatabaseReader` or `DatabaseWriter` inside an action. Actions don't have access to the database. Use `runQuery` and `runMutation`.
- Below is an example of the syntax for an action:

`confect/jobs.spec.ts` (excerpt)

```ts
  .addFunction(
    FunctionSpec.publicAction({
      name: "exampleAction",
      returns: () => Schema.Null,
    }),
  )
```

`confect/jobs.impl.ts` (excerpt)

```ts
const exampleAction = FunctionImpl.make(
  databaseSchema,
  jobs,
  "exampleAction",
  () =>
    Effect.gen(function* () {
      yield* Effect.log("This action does not return anything");
      return null;
    }),
);
```

## Scheduling guidelines

### Cron guidelines

- Only use `CronJob.make(name, schedule, ref, args?)` from `@confect/server` to schedule cron jobs. The schedule is a `Cron` from `effect/Cron` (it becomes `crons.cron`) or a `Duration` from `effect/Duration` (it becomes `crons.interval`). Do NOT use the `crons.hourly`, `crons.daily`, or `crons.weekly` helpers; Confect does not offer them.
- A `Cron` schedule has minute granularity and no seconds field. A `Duration` interval must be a whole number of seconds, minutes, or hours. Convex runs schedules in UTC.
- `CronJob.make` takes a ref from `confect/_generated/refs`. Do NOT try to pass the function directly.
- Define crons in `confect/crons.ts`: default-export `CronJobs.make()` with one `.add(CronJob.make(...))` per job. Codegen writes `convex/crons.ts`. For example,

`confect/crons.ts`

```ts
import { CronJob, CronJobs } from "@confect/server";
import * as Cron from "effect/Cron";
import * as Duration from "effect/Duration";

import refs from "./_generated/refs";

export default CronJobs.make()
  .add(
    CronJob.make(
      "purge completed tasks",
      Cron.parseUnsafe("0 3 * * *"),
      refs.internal.tasks.purgeCompleted,
    ),
  )
  .add(
    CronJob.make(
      "post hourly status",
      Duration.hours(1),
      refs.internal.messages.postSystem,
      { channel: "status", body: "Still running." },
    ),
  )
  .add(
    // Run refs.internal.jobs.empty every two hours.
    CronJob.make("run empty job", Duration.hours(2), refs.internal.jobs.empty),
  );
```

- `confect/crons.ts` holds only the cron jobs. Declare the function that a cron calls in a group spec, as with any other function (here, `internalAction` `empty` in `confect/jobs.spec.ts`).
- If a cron calls an internal function, always use `refs.internal` from `confect/_generated/refs`.

## Testing guidelines

- Use `@confect/test` with `convex-test`, `vitest`, and `@effect/vitest` to test Confect functions. Install `@confect/test` at the same version as the other `@confect/*` packages. Always install the latest versions of `convex-test`, `vitest`, and `@edge-runtime/vm`. Configure vitest with `environment: "edge-runtime"` in `vitest.config.ts`, as Convex advises. The examples in this file were verified with `environment: "node"`. They were not run with `edge-runtime`. If a test fails under `edge-runtime`, use `"node"`. Inline `convex-test` in the Vitest config (`server.deps.inline: ["convex-test"]`). Vitest treats `*.spec.ts` as test files by default, so exclude `**/*.spec.ts`; Confect spec files use that suffix.

Test files go outside `convex/` (for example in `test/`). Codegen removes `.ts` files under `convex/` that are not Confect group modules. Build the test layer with `TestConfect.layer(confectSchema, convexSchema, modules)`. You must pass a module map from `import.meta.glob` that includes the `convex/` modules and `convex/_generated/`:

`test/TestConfect.ts`

```ts
/// <reference types="vite/client" />

import { TestConfect as TestConfect_ } from "@confect/test";

import convexSchema from "../confect/_generated/convexSchema";
import confectSchema from "../confect/_generated/schema";

export const TestConfect = TestConfect_.TestConfect<typeof confectSchema>();

// A new test database each time the layer is built.
export const layer = TestConfect_.layer(
  confectSchema,
  convexSchema,
  import.meta.glob(["../convex/**/*.ts", "!../convex/**/*.d.ts"]),
);
```

`test/messages.test.ts` (excerpt)

```ts
  it.effect("an action reads and writes through the runners", () =>
    Effect.gen(function* () {
      const confect = yield* TestConfect.TestConfect;

      yield* confect.mutation(refs.internal.messages.postSystem, {
        channel: "general",
        body: "hi",
      });
      const echoed = yield* confect.action(refs.public.messages.echoLatest, {
        channel: "general",
      });
      expect(echoed).not.toBeNull();

      const bodies = yield* confect.query(refs.internal.messages.latestBodies, {
        channel: "general",
      });
      expect(bodies).toEqual(["echo: hi", "hi"]);
    }).pipe(Effect.provide(TestConfect.layer)),
  );
```

The `modules` argument is required so convex-test can discover and load function files. The `/// <reference types="vite/client" />` directive is needed for TypeScript to recognize `import.meta.glob`.

- Only add the `/// <reference types="vite/client" />` directive at the top of files that call `import.meta.glob`; do NOT add it to other files.
- Do NOT add a `compilerOptions.types` allowlist to `tsconfig.json` for type packages you have not installed (e.g. `"node"` without `@types/node`, or `"vite/client"` without vite). Any unresolved entry in `types` fails typechecking with TS2688. Leave `types` unset unless a package genuinely requires it and is installed.
- A component that the code under test calls must also be registered with `convex-test`.
- Use `withIdentity`, `run`, `fetch`, `finishInProgressScheduledFunctions`, and `finishAllScheduledFunctions` on the `TestConfect` service. See E16 in `confect-patterns.md`.

## File storage guidelines

- The `storage.getUrl(id)` call on the `Storage` service returns a signed URL (a `URL`) for a given file. It fails with `BlobNotFoundError` if the file doesn't exist. Return it with `Schema.URLFromString`.
- Do NOT use the deprecated `ctx.storage.getMetadata` call for loading a file's metadata. The Confect `Storage` service does not expose it.

Instead, query the `_storage` system table. Use `reader.table("_storage").get(storageId)` with a `GenericId.GenericId("_storage")` argument. A `_storage` document has `_id`, `_creationTime`, `contentType` (optional), `sha256`, and `size`. For example:

`confect/files.spec.ts` (excerpt)

```ts
  .addFunction(
    FunctionSpec.publicQuery({
      name: "metadata",
      args: () => ({ storageId: GenericId.GenericId("_storage") }),
      returns: () =>
        Schema.NullOr(
          Schema.Struct({
            size: Schema.Finite,
            contentType: Schema.optionalKey(Schema.String),
          }),
        ),
    }),
  )
```

`confect/files.impl.ts` (excerpt)

```ts
// File metadata lives in the _storage system table.
// Do not use the deprecated storage getMetadata call.
const metadata = FunctionImpl.make(
  databaseSchema,
  files,
  "metadata",
  ({ storageId }) =>
    Effect.gen(function* () {
      const reader = yield* DatabaseReader;
      const stored = yield* reader.table("_storage").get(storageId);

      return {
        size: stored.size,
        ...(stored.contentType === undefined
          ? {}
          : { contentType: stored.contentType }),
      };
    }).pipe(
      Effect.catchTag("GetByIdFailure", () => Effect.succeed(null)),
      Effect.orDie,
    ),
);
```

- Convex storage stores items as `Blob` objects. You must convert all items to/from a `Blob` when using Convex storage. `storage.store(blob)` and `storage.get(id)` work only in actions and HTTP actions.
