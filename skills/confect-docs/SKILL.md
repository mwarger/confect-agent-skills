---
name: confect-docs
description: "Pull version-current Confect and Convex docs for the versions this project uses — pin the installed versions, fetch page-as-markdown or check node_modules types, freshness hierarchy — instead of writing a possibly-stale Confect or Convex API from memory."
---

<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Pull version-current Confect and Convex docs

confect-expert carries baked, plugin-versioned knowledge — excellent for stable idioms, but it goes stale exactly where it hurts: a component that gained a new export, a CLI flag that changed, an API renamed between versions. This capability is the freshness discipline layered on top: pin to the project's real version, fetch the live page cheaply as markdown, and never write an unfamiliar API from memory when the current source is one fetch away.

Two layers need docs. Convex is the platform. Confect is the Effect layer on top. The Convex guidelines in the project's `convex/_generated/ai/guidelines.md` when that file exists (`npx convex ai-files install` creates it) and the verified Confect table in `references/confect-patterns.md` are the baked knowledge. Use this skill when they may be out of date.

## Workflow

1. PIN the version: read the installed `convex` version (`node -p "require('./node_modules/convex/package.json').version"` or `package.json`). Also read the versions of `@confect/core`, `@confect/server`, `@confect/cli`, `@confect/test`, and `effect`, and of any `@convex-dev/*` components in play. The docs you trust must match THESE versions — version skew is the single largest source of wrong Convex and Confect code.
2. FRESHNESS HIERARCHY (cheapest-correct first, the Supabase-taught order):
   (a) if a served docs tool / MCP `search_convex_docs` is available, use it for Convex (it returns version-scoped, reranked answers sized to the context window);
   (b) else fetch the specific docs page as MARKDOWN — request `docs.convex.dev/<path>` for Convex, or the Confect docs (https://confect.dev), and prefer a `.md`/markdown form when the site serves one (far fewer tokens than HTML), or the component's README at the pinned version;
   (c) only then fall back to a general web search, and treat its version as unverified.
   Do NOT skip to writing the API from memory when currentness is in doubt.
3. VERIFY against the installed package when it matters: for a component export you're unsure exists, check `node_modules/@convex-dev/<x>/` (its `package.json` `exports`, its `.d.ts`). For a Confect or Effect API, check `node_modules/@confect/<pkg>/` (`src/` and `dist/*.d.ts`) and `node_modules/effect/`. The installed types are the ground truth for THIS version, more authoritative than any doc. The Confect docs can describe a newer version than the one installed. When the docs and the installed source differ, the source wins.
4. USE the fetched fact narrowly: apply the current signature/flag, cite where it came from (page + version), and hand the actual code back to confect-expert to write idiomatically. confect-docs supplies the fresh fact; confect-expert supplies the idiom.
5. On a version-mismatch build error (an export/flag that 'should' exist but doesn't): treat it as a currentness question — pin the version, fetch the current API, and correct — rather than guessing a different spelling.

## Rules

- Never write an unfamiliar or possibly-renamed Convex, Confect, or component API from model memory when currentness is in doubt — pin the version and fetch the current source first.
- The installed package's own `exports`/`.d.ts` in node_modules is the ground truth for this version — more authoritative than any doc page.
- Follow the freshness hierarchy: served docs tool → page-as-markdown / pinned README → general web (unverified) — cheapest-correct first, fewest tokens.
- Prefer markdown over HTML doc pages — far fewer tokens for the same content.
- Supply the fresh FACT; hand idiomatic code back to confect-expert. This is a freshness layer, not a replacement for the baked knowledge.
- A version-mismatch build error is a currentness question, not a spelling guess — re-pin and re-fetch.
- Platform facts (limits, index rules, transactions) come from the Convex docs. Confect API names come from the Confect package source.
