# Ledger: convex-agent (SKILL.md, 25 lines)

Source file: skills/convex-agent/SKILL.md. Directory name kept; a later step renames it.
Code content: API mentions only (component, models, actions). No code samples upstream.

| Source | Action | Reason |
| --- | --- | --- |
| Frontmatter `name: convex-agent` | update | `confect-agent`. |
| Frontmatter `description` | update | Confect app with a `confect/` directory. Agent, threads, tool calls, vector search kept. |
| GENERATED comment | keep | Unchanged upstream line. |
| (new) "Adapted to Confect" comment | add | Same marker as the adapted skills. |
| `# Add an AI agent / RAG backend` | keep | Unchanged. |
| Intro paragraph (agent component, gateway, no key) | keep | Platform facts. Unchanged. |
| (new) Paragraph: component is plain Convex; call client with raw ctx; pointers to E6, E10, E11, E12 | add | Confect does not wrap components (patterns: "Component client calls"). |
| Workflow 1: install packages, add component to convex.config.ts | update | Kept. Added: you own that file; run codegen so `components.agent` appears. |
| Workflow 2: define agent with `convexGateway`, no key, convex 1.45+, paid plan | update | All platform facts kept. Added: pass `components.agent` from the Confect registry. |
| Workflow 3: create threads, stream messages, persist history | update | Kept. Added: Confect action, `yield* ActionCtx`, `Effect.promise`, identity from `Auth`. |
| Workflow 4: RAG, embeddings key via `env` micro power | update | Kept. Added `.vectorIndex(...)` on a table, `VectorSearch` service in an action only (E1, E9). |
| Workflow 5: gateway fallback with provider key via `env` | update | Kept. Added `Config.String("NAME")` to read the key (E10). |
| (new) Workflow 6: client message queries as plain Convex functions | add | The agent React hooks need exported component queries. Confect does not wrap them. Use `FunctionSpec.convex*` (E12). |
| (new) Workflow 7: codegen then tsc | add | Patterns workflow rule. |
| (new) Code sample: `confect/assistant.ts` | add | Shows the Agent with the Confect components registry (E6 pattern). Neutral domain. Not compiled. |
| (new) Code sample: impl `ask` with `ActionCtx` | add | Shows raw ctx in `Effect.promise` (E6, E12 patterns). Not compiled. |
| `## Rules` | keep | Heading. |
| Rule: default to gateway, fall back to provider key | keep | Unchanged. |
| Rule: never expose provider key client-side | keep | Unchanged. |
| Rule: run model calls in actions ('use node' if needed) | update | Kept. `"use node"` became a Node group (`GroupSpec.makeNode()`, `publicNodeAction`, `internalNodeAction`). Added: Node group holds only Node actions; Node actions use Effect's default provider (E10, E11). |
| Rule: persist threads/messages in Convex | keep | Unchanged. |
| (new) Rule: raw ctx service in `Effect.promise`; registry from `confect/_generated/components` | add | Patterns table rows for components. |
| (new) Rule: auth check before calling the component | add | Patterns E6 closing paragraph and "Components" platform rule. |
| (new) Rule: `internalAction` for model calls only your code starts | add | Patterns "Platform rules": internal functions for own-code calls. |
| (new) Rule: each action `runQuery` or `runMutation` is a separate transaction | add | Patterns E6 platform fact. |
| (review fix) Workflow 5: Confect gateway client for direct model calls | add | `AiGatewayLanguageModel`, `AiGatewayLanguageClient`, `AiGatewayDisabled`, `AiGatewayUnavailable`, `AiGatewayError`. |
| (review fix) Section `Which gateway path` | add | Says when to use `convexGateway` and when to use `AiGatewayLanguageModel`. |
| (review fix) Spec code sample and Bad and Good impl labels | add | Example is complete: imports, spec with `error` schema, impl, group. |

## Review fixes

- Pointers use the full path. Workflow 3 and the component rule use `Effect.tryPromise` mapped to a branded `Schema.TaggedError`. New workflow step 5 and section `Which gateway path` name `AiGatewayLanguageModel`, `AiGatewayLanguageClient` and its typed errors (verified in `@confect/server` 10.0.0 source). The `ask` example is now spec plus impl with a Bad and a Good form. The Good form reads the identity with `Auth`, sets `userId` on the thread, and maps a model failure to `AssistantUnavailable`. Steps 6 and 7 renumbered to 7 and 8.

## Open questions

- The two code samples use the `@convex-dev/agent` public API (`new Agent`, `createThread`, `generateText`) from memory. They were not compiled against the installed component version. Check them before release.
- `"provider/model"` is a placeholder, as upstream. No real model id was invented.
- The new `ask` spec and impl were not compiled in this step. The package set cannot be installed here. Compile them before release.
- `AiGatewayLanguageModel` usage is described, not shown in code. Add a compiled example.
- The six skills have `name: confect-*` but the directory is still `convex-*`. The cross-references to confect-reviewer, confect-authz and confect-expert do not resolve yet. The later rename step must fix both together. Not changed here on purpose.
