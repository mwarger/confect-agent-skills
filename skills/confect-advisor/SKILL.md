---
name: confect-advisor
description: "Read the deployment's 72h insights (read limits, OCC contention) for a Confect app (Effect on Convex), root-cause each event in the specs and impls, report evidence-backed perf/cost findings with Confect fixes. Use when a project has a confect/ directory."
---

<!-- GENERATED from convex-agents content/capabilities/convex-advisor.json — do not edit by hand. -->
<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Live-deployment advisor

Static review guesses; the deployment KNOWS. The official Convex MCP ships an `insights` tool with typed 72h health events per function — documentsReadLimit / bytesReadLimit (hard limit hits), documentsReadThreshold / bytesReadThreshold (approaching), occFailedPermanently / occRetried (write contention) — each carrying evidence (table_name, bytes_read, documents_read, occ document id + retry count). The advisor turns each event into a root-caused finding by reading the flagged function's actual code, and emits findings on the findings bus (specs/finding.schema.json) so fixers can be dispatched and launch-readiness can score.

The deployment runs the generated `convex/` modules. Insight events name those Convex functions. The code that you read is in `confect/`. Read `skills/confect/references/confect-patterns.md` for the Confect form of each fix. Read `skills/confect/references/guidelines.md` for the Convex platform rules.

## Workflow

1. GUARD: run deploy-guard step 0-1 — identify + announce the deployment being read. Reading insights/logs on prod is allowed read-only; never enable mutating prod access for an advisory pass.
2. GATHER (deterministic, via the official Convex MCP): `status` → deployment selector; `insights` → the typed 72h events; `tables` → schema + row counts; `functionSpec` → the public/internal surface. The `insights` tool is only available on cloud dev/prod deployments when logged in as a user (not on previews or deploy-key-scoped contexts) and needs ~72h of traffic; if it returns nothing or is unavailable, say so and fall back to offering confect-reviewer — do NOT invent findings.
3. MAP each event to its Confect source. A Convex function id such as `notes:listMine` is the group path and the function name. Open `confect/notes.spec.ts` for the spec. Open `confect/notes.impl.ts` for the `FunctionImpl.make(databaseSchema, notes, "listMine", ...)` call. The `functionSpec` tool shows the generated Convex surface, not the Confect specs. A function in a middleware chain also runs the middleware reads. Include them in the root cause.
4. ROOT-CAUSE each insight event by reading the flagged function's code:
   - bytesReadThreshold/Limit or documentsReadThreshold/Limit → look for these causes on the named table:
     - `.collect()` on a table that can grow.
     - An index range that is too wide, for example a `.index("by_creation_time")` scan.
     - A predicate that runs after the scan. Examples are `.stream()` with `Stream.filter` and the `filter` argument of `paginate`.
     - No pagination.

     Confect has no `.filter()` on queries. The fix is an index plus an index range in `.index(name, (q) => ...)`. Other fixes are `.take(n)` and `publicPaginatedQuery` with `.paginate(paginationOpts)` (confect-expert patterns). For counting shapes, use an aggregate component.
   - occRetried / occFailedPermanently → look for read-modify-write hotspots on the named document (shared counters, status toggles); the fix is @convex-dev/sharded-counter, narrowing the read set, or moving contention to a workpool. A `reader.table(t).get(id)` or an index read in the mutation puts that document in the read set.
   - repeated failures in `logs` (status: failure) → classify: crash loop in a cron, argument decode rejections, unhandled error shapes. In Confect, a failure that is not in the spec's `error` schema becomes a defect. The caller sees a generic failure. Look for expected failures that are missing from the `error` schema.
5. EMIT findings per specs/finding.schema.json: class perf/correctness/cost, severity from the insight kind (limit hits = high, thresholds = med, retried = med, permanent OCC failure = high), locus {kind: deployment, functionId, tableName}, evidence {kind: insight-event, detail: the raw event}, confidence: confirmed (the event happened — it is not a hypothesis), fixCapability + autofixable where the repair is mechanical.
6. REPORT: findings ranked by severity, each with (a) the runtime evidence in one line ('messages:list read 4.2MB from messages 31× yesterday'), (b) the code-level root cause with file:line (the `confect/*.impl.ts` line, not the generated module), (c) the concrete fix and which capability applies it. Offer to apply fixes; apply only on confirmation. After a fix, run `npx confect codegen` and `tsc --noEmit`. Then re-run `insights` after traffic to verify the trend, or re-run the static check immediately.
7. Scope discipline: this is a health/perf/cost pass. Route authz findings to confect-authz, code-idiom findings to confect-reviewer, error triage to sentinel — emit a pointer finding rather than duplicating their work.

```ts
// Bad: reads every row, then drops most of them. The rows read do not go down.
const rows = yield* reader.table("messages").index("by_creation_time").take(5000);
return rows.filter((row) => row.channel === channel);
```

```ts
// Good: the channel is in the index range, and the result is bounded.
return yield* reader
  .table("messages")
  .index("by_channel", (q) => q.eq("channel", channel), "desc")
  .take(50);
```

## Rules

- Evidence-not-vibes: every finding cites a real insight event, log line, or table stat — if the deployment has no evidence, the advisor has no findings (offer confect-reviewer instead).
- Read-only by construction: an advisory pass never mutates any deployment and never enables prod mutation flags (deploy-guard discipline applies).
- Root-cause in the code before reporting: an insight event names the symptom; the finding must name the line and the mechanism.
- Emit on the findings bus (specs/finding.schema.json), confidence: confirmed — runtime events are facts, not hypotheses.
- Severity from the event kind: limit-hit / permanent-OCC-failure = high; threshold / retried = med.
- Stay in lane: perf/cost/health only — hand authz to confect-authz, style to confect-reviewer, error triage to sentinel.
- Prefer component fixes over hand-rolls when they match (sharded-counter for OCC on counters, aggregate for count scans) — same bias as suggest. Call a component client with the raw ctx service (`MutationCtx`), in the same mutation as the table write.
- Report the Confect source (`confect/<group>.impl.ts`), not the generated `convex/<group>.ts` module. Never fix a generated file.
- A `.stream()` with `Stream.filter` or the `filter` argument of `paginate` does not reduce the rows read. Do not offer it as a read-limit fix.
