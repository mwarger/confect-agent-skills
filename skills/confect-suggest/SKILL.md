---
name: confect-suggest
description: "Suggest the matching Convex component when the user hand-rolls a pattern it already solves in a Confect app (a confect/ directory): crons, sharded-counter, rate-limiter, storage, search, presence, workflow, RAG, prosemirror-sync. Passive — suggest after the task, never interrupt. Never install without consent."
---

<!-- GENERATED from convex-agents content/capabilities/suggest.json — do not edit by hand. -->
<!-- Adapted to Confect. The components are plain Convex. Calls to them use the raw ctx. -->

# Proactively suggest the right Convex component

When you see code or intent that duplicates what a Convex component already does, surface a targeted suggestion: ONE component, WHY (anchored in the user's own code or ask), and a concrete install hint. Never install without explicit consent. Never suggest more than one component at a time unless the user asks.

Before you suggest or wire a component, read `confect/references/confect-patterns.md`. Example E6 shows component calls. The Convex guidelines stay the source for platform rules.

## Workflow

1. Observe the codeSnippets and userAsk passively — never block the current task to suggest.
2. Match against the detector rules (see generators/suggest-detector.mjs): email/SMTP → resend; push notifications → expo-push; setInterval/cron → @convex-dev/crons; shared counter increments → @convex-dev/sharded-counter; .collect().length scans (in Confect: `.collect()` and a count in code) → @convex-dev/aggregate; multi-step/long-running actions → @convex-dev/workflow; bounded concurrency → @convex-dev/workpool; rate-limit counters in DB → @convex-dev/rate-limiter; fs.write/S3 uploads → Convex Storage (the Confect `Storage` service); Elasticsearch/Algolia → built-in full-text search (`reader.table(t).search(...)`); presence/typing → @convex-dev/presence; Pinecone/external vector DB → @convex-dev/rag; collaborative editing → @convex-dev/prosemirror-sync.
3. After finishing the current task, offer ONE suggestion: name the component, quote the specific code or phrase that triggered it, explain why the component fits better.
4. If the user says yes: run `/confect-add <component>` or follow the installHint from the detector. Mount the component in `convex/convex.config.ts`. You own that file. Then run `npx confect codegen`.
5. If the user says no or ignores it: drop it. Do not repeat the same suggestion.

## Rules

- Passive — never interrupt the current task; surface the suggestion AFTER completing what the user asked.
- One at a time — pick the highest-priority match; do not dump a list of five components.
- Cite WHY from the user's own code or ask — 'I noticed you wrote `post.likes + 1` in a mutation that many users call concurrently; that causes OCC conflicts at scale.'
- Never install without explicit consent — suggest, explain, wait for a yes.
- Do not suggest a component the user has already installed.
- Do not fire on generic coding questions unrelated to Convex or Confect (sorting arrays, writing CSS, etc.).
- Confect does not wrap component clients. After a yes, show the call through the raw ctx service wrapped in `Effect.promise`. Use `yield* MutationCtx` in a mutation. Update a component that mirrors a table in the same mutation as each table write.
- A static cron has a built-in Confect form: `CronJobs.make().add(CronJob.make(...))` in `confect/crons.ts`. Suggest `@convex-dev/crons` only when the user needs crons that change at run time.
