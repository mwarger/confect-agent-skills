---
name: confect-cost
description: "Preview spend for a Confect app (Effect on Convex) — rank functions by bytes/documents-read × call-volume from insights, project each cost driver's growth curve, name the cheapest Confect fix; confirm-cost for paid actions. Use when a project has a confect/ directory."
---

<!-- GENERATED from convex-agents content/capabilities/convex-cost.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Preview what this app will cost

Cost surprises come from a handful of functions reading far more data than anyone realized — the same read-heavy patterns confect-advisor flags for perf, seen through the money lens. This capability makes spend legible: it reads the deployment's own bytes/documents-read evidence, attributes it to the functions driving it, projects how it grows with traffic, and names the cheapest fix. It also carries the confirm-cost discipline (Supabase's structural consent for paid actions): before anything metered, state the price and get an explicit yes.

Confect does not change what Convex bills for. The deployment runs the generated `convex/` modules. Insight events name those Convex functions. The code is in `confect/`. Read `skills/confect/references/confect-patterns.md` for the Confect form of each fix. Read `skills/confect/references/guidelines.md` for the Convex platform rules.

## Workflow

1. GUARD: deploy-guard — a cost read is read-only over dev/prod (insights is cloud+user-auth only; not previews). Announce the deployment.
2. GATHER the spend evidence via the official MCP.
   - `insights` gives the bytes-read and documents-read events. This is the direct cost signal. Convex bills on function calls and bandwidth.
   - `tables` gives row counts. The size of a table bounds its scan cost.
   - `functionSpec` gives the surface.
   - Map each Convex function id (for example `notes:listMine`) to `confect/notes.impl.ts`.
   - If there is no usage or traffic yet, say so. Estimate from the query SHAPES instead. An unbounded `.collect()` is a future cost even with zero traffic today. A wide `.index("by_creation_time")` scan on a table projected to grow is the same.
3. ATTRIBUTE: rank functions by bytes/documents read per call × observed (or asked-about) call volume — the product is the cost driver, not either alone. A cheap-per-call function called constantly can outweigh an expensive rare one; show both factors.
4. PROJECT: state how the top drivers scale — a full-table `.collect()` grows LINEARLY with the table (cost compounds as data accumulates); an indexed `.take(n)` stays flat. Give the user the shape of the curve ('this is O(table size) per call — fine at 1k rows, a bill at 1M'), not a false-precision dollar figure.
5. NAME THE CHEAPEST FIX per driver. Options:
   - An index plus an index range in `.index(name, (q) => ...)`, instead of a scan.
   - `.paginate` (through `publicPaginatedQuery`) or `.take`, instead of `.collect`.
   - An aggregate component for counts.
   - Caching a hot read.

   Emit the fix as a cost-class finding on the bus. The evidence is the insight event and the projected growth. Point at confect-expert or confect-advisor for the actual change. Confect has no `.filter()` on queries. A `.stream()` with `Stream.filter` does not reduce the rows read. It is not a cost fix.
6. CONFIRM-COST for paid actions: if the flow includes anything metered (a domain purchase, cloud provisioning, a plan change), STATE the price and recurrence explicitly and get an explicit yes BEFORE proceeding — never let a paid action happen as a side effect (the cost-confirm gate).
7. REPORT: the current cost drivers ranked, each with its evidence + growth shape + fix, and a plain bottom line ('your spend is dominated by messages:list reading the whole table every call; index it and it drops ~100x'). Honest precision: Convex pricing changes and depends on plan — give relative/shape guidance and cite the pricing page for absolute numbers rather than inventing a dollar total.

```ts
// Bad: collect() reads every row for one author. An author can have a growing number of notes. Cost grows with that number.
const rows = yield* reader.table("notes").index("by_authorId", (q) => q.eq("authorId", user._id)).collect();
```

```ts
// Good: the read is bounded. Cost stays flat as the author's notes grow.
const rows = yield* reader
  .table("notes")
  .index("by_authorId", (q) => q.eq("authorId", user._id), "desc")
  .take(50);
```

## Rules

- Cost = data-read-per-call × call-volume — always show both factors; a cheap function called constantly can cost more than an expensive rare one.
- Read the deployment's own insights/bytes-read evidence for spend; with no traffic yet, price the query SHAPES (a scan on a growing table is a future cost).
- Give the growth CURVE, not false-precision dollars: O(table) scans compound as data accumulates; indexed access stays flat. Cite the pricing page for absolute figures.
- Every cost driver names its cheapest fix and emits a cost-class finding on the bus pointing at the fixer (confect-expert/advisor).
- Confirm-cost for any metered/paid action: state the price + recurrence and get an explicit yes BEFORE it happens — never as a side effect.
- Read-only over dev/prod (deploy-guard); insights is cloud+user-auth only. Cost composes confect-advisor's evidence but frames it as money, not latency.
- A clock read in a query impl (`Date.now()` or `Clock.currentTimeMillis`) makes Convex evict the query from the cache. Each call then pays for a fresh read. Pass the time as an argument. Confect 10.0.0 does not stub `Date.now()`.
- Report the Confect source (`confect/<group>.impl.ts`), not the generated `convex/<group>.ts` module. Never fix a generated file.
- Each `runQuery` and `runMutation` call from an action is a separate transaction. Many calls from one action add cost. Keep them few.
