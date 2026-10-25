---
name: confect-agent
description: "Add an AI agent / RAG backend (@convex-dev/agent) to a Confect app (Effect on Convex). Use when the project has a confect/ directory and the user wants an in-app AI agent, threads, tool calls, or vector search."
---

<!-- GENERATED from convex-agents content/capabilities/agent.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Add an AI agent / RAG backend

Install @convex-dev/agent for durable threads, message history, tool-calls, and vector search/RAG — the backend for an in-app AI agent. Call models through the Convex AI Gateway by default: Convex holds the provider credentials, so there is no LLM key to obtain, store, or rotate.

The agent component and its client are plain Convex code. Confect wraps your own functions, not the component. Call the component client with the raw ctx service inside an Effect. Read `skills/confect/references/confect-patterns.md` (sections E6, E10, E11, E12) and `skills/confect/references/guidelines.md` first.

## Workflow

1. Install @convex-dev/agent + @convex-dev/ai-sdk-provider; add the agent component to `convex/convex.config.ts`. You own that file. Run `npx confect codegen` so that `components.agent` appears in `confect/_generated/components`.
2. Define the agent (tools, instructions) with `languageModel: convexGateway("provider/model")` — no API key needed (needs convex 1.45+ on a Convex Cloud deployment, paid plan). Pass `components.agent` from `confect/_generated/components` to the `Agent` constructor.
3. Create threads + stream messages; persist history in Convex. Make each model call from a Confect action. Get the raw ctx with `yield* ActionCtx`. Pass it to the agent client inside `Effect.tryPromise`. Map a failure to a branded `Schema.TaggedError` that is in the spec `error` schema. Read the identity with the `Auth` service, never from an argument.
4. For RAG: embed docs into a vector index and retrieve in the tool. The gateway does not serve embeddings yet, so store the embedding provider's key via the `env` micro power. Declare the index with `.vectorIndex(...)` on a table in `confect/tables/`. Search it with the `VectorSearch` service, in an action only.
5. For a direct model call without the agent component, use the Confect gateway client. `AiGatewayLanguageModel` (`model`, `make`, `layer`) takes a `provider/model` id. `AiGatewayLanguageClient` provides the model client. It needs an Effect `HttpClient`. Typed errors are `AiGatewayDisabled`, `AiGatewayUnavailable`, and `AiGatewayError`. Use `convexGateway` as the `languageModel` of the `Agent` class. See "Which gateway path" below.
6. Only if the gateway is unavailable (free plan, self-hosted, local backend): call the provider SDK with a key stored via the `env` micro power. Read the key with `Config.String("NAME")`.
7. When the client UI needs the component's message queries (for example, a message list hook), expose them as plain Convex functions in `confect/<group>.ts`. Name them in the spec with the `FunctionSpec.convex*` constructors. Confect does not wrap these functions. They use Convex validators.
8. Run `npx confect codegen`, then `tsc --noEmit`.

`confect/assistant.ts`

```ts
import { Agent } from "@convex-dev/agent";
import { convexGateway } from "@convex-dev/ai-sdk-provider";

import { components } from "./_generated/components";

// Good: use the Confect components registry, not convex/_generated/api.
export const assistant = new Agent(components.agent, {
  name: "Notes assistant",
  languageModel: convexGateway("provider/model"),
  instructions: "Answer questions about the user's notes.",
});
```

`confect/assistantChat.spec.ts`

```ts
import { FunctionSpec, GroupSpec } from "@confect/core";
import * as Schema from "effect/Schema";

export class NotSignedIn extends Schema.TaggedError<NotSignedIn>()(
  "NotSignedIn",
  {},
) {}

export class AssistantUnavailable extends Schema.TaggedError<AssistantUnavailable>()(
  "AssistantUnavailable",
  { message: Schema.String },
) {}

export default GroupSpec.make().addFunction(
  FunctionSpec.publicAction({
    name: "ask",
    args: () => ({ prompt: Schema.String }),
    returns: () => Schema.String,
    // Expected failures are in the error schema. The caller gets them decoded.
    error: () => Schema.Union([NotSignedIn, AssistantUnavailable]),
  }),
);
```

`confect/assistantChat.impl.ts`

```ts
import { FunctionImpl, GroupImpl } from "@confect/server";
import * as Effect from "effect/Effect";
import * as Layer from "effect/Layer";

import { ActionCtx, Auth } from "./_generated/services";
import databaseSchema from "./_generated/schema";
import { assistant } from "./assistant";
import assistantChat, {
  AssistantUnavailable,
  NotSignedIn,
} from "./assistantChat.spec";

// Bad: no auth check, no user on the thread, and a rejection becomes a defect.
const askBad = FunctionImpl.make(databaseSchema, assistantChat, "ask", ({ prompt }) =>
  Effect.gen(function* () {
    const ctx = yield* ActionCtx;
    const result = yield* Effect.promise(async () => {
      const { thread } = await assistant.createThread(ctx, {});
      return thread.generateText({ prompt });
    });
    return result.text;
  }),
);

// Good: check the identity first. Own the thread by user. Map a model failure to a typed error.
const ask = FunctionImpl.make(databaseSchema, assistantChat, "ask", ({ prompt }) =>
  Effect.gen(function* () {
    const auth = yield* Auth;
    // The only failure is NoUserIdentityFoundError. Map it to the typed error.
    const identity = yield* auth.getUserIdentity.pipe(
      Effect.mapError(() => new NotSignedIn()),
    );
    // The agent client takes the raw Convex ctx. Run model calls in an action.
    const ctx = yield* ActionCtx;
    const result = yield* Effect.tryPromise({
      try: async () => {
        const { thread } = await assistant.createThread(ctx, {
          userId: identity.tokenIdentifier,
        });
        return thread.generateText({ prompt });
      },
      catch: (cause) => new AssistantUnavailable({ message: String(cause) }),
    });
    return result.text;
  }),
);

export default GroupImpl.make(databaseSchema, assistantChat).pipe(
  Layer.provide(ask),
  GroupImpl.finalize,
);
```

The `ask` function is public. It checks the identity before it calls the model. Without this check, any caller can start metered model calls. Recover from a typed model failure with `Effect.catchTag("AssistantUnavailable", ...)`.

## Which gateway path

- Use `convexGateway` with the `Agent` class. The agent component and its client take an AI SDK model. Confect does not wrap the component.
- Use `AiGatewayLanguageModel` for a direct model call outside the agent component. It is an Effect AI model with typed errors. Provide `AiGatewayLanguageClient.layer` and an Effect `HttpClient` in one combined layer with `Layer.provideMerge`.
- Both paths use the same Convex AI Gateway. Both need the same plan and convex version.

## Rules

- Default to the Convex AI Gateway (`convexGateway` from @convex-dev/ai-sdk-provider) for model calls; fall back to a provider key in Convex env only where the gateway is unavailable (free plan, self-hosted, local backend).
- Never expose a provider API key client-side; when one is needed (embeddings, gateway fallback), keep it in Convex env via the `env` micro power.
- Run model calls in actions. Use a Node group (`GroupSpec.makeNode()` with `FunctionSpec.publicNodeAction` or `internalNodeAction`) if the SDK needs Node. A Node group holds only Node actions. Node actions read env vars with Effect's default provider.
- Persist threads/messages in Convex for durability + reactivity.
- Call the component client with the raw ctx service (`ActionCtx`, `MutationCtx`, or `QueryCtx`) inside `Effect.tryPromise`. Map the failure to a `Schema.TaggedError` in the spec `error` schema. A rejection inside `Effect.promise` becomes a defect. Get the component reference from `confect/_generated/components`.
- Do the auth check in your own function before you call the component. A component function is not callable from the client.
- Use `FunctionSpec.internalAction` for a model call that only your code starts (a scheduled job, for example). Schedule it with `refs.internal.*`.
- Each `runQuery` and `runMutation` call from an action is a separate transaction. Keep these calls few.
