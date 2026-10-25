---
name: confect-crons
description: "Add recurring scheduled jobs (crons) to a Confect app (Effect on Convex). Use for confect/crons.ts, CronJobs, CronJob, and refs.internal targets."
---

<!-- Adapted to Confect. Platform rules come from the Convex guidelines. -->

# Add scheduled jobs (crons)

Define recurring jobs in confect/crons.ts targeting internal functions, with sane intervals and idempotent handlers.

Read `references/confect-patterns.md` (example E15) in the confect skill before you write code. Read the project's `convex/_generated/ai/guidelines.md` when that file exists (`npx convex ai-files install` creates it) for the Convex platform rules.

## Workflow

1. Create confect/crons.ts. Default-export `CronJobs.make()`. Add each job with `.add(CronJob.make(...))`.
2. Schedule internal functions (`refs.internal.*`, never `refs.public.*`) at the right interval. Declare the target with `FunctionSpec.internalMutation` or `FunctionSpec.internalAction` in a spec.
3. Make handlers idempotent (safe to re-run); keep each run small.
4. Run `npx confect codegen`, then `tsc --noEmit`.
5. Verify the job appears in the dashboard schedule.

```ts
// Bad: a raw Convex cron file, a public target, and a time zone.
// convex/crons.ts
import { cronJobs } from "convex/server";
const crons = cronJobs();
crons.interval("post status", { hours: 1 }, api.messages.postSystem, {});
export default crons;

// Bad: a Cron with a time zone throws when the module loads.
Cron.parseUnsafe("0 3 * * *", "America/New_York");

// Good: confect/crons.ts
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
  );
```

## Rules

- Schedule refs.internal.* functions, never refs.public.*.
- Keep cron handlers small + idempotent.
- Don't poll tight intervals for things a subscription can push.
- A `Cron` schedule has minute granularity and no seconds field. Convex runs schedules in UTC.
- A `Duration` interval must be a whole number of seconds, minutes, or hours.
- Do not use `convex/crons.ts` or `cronJobs()`. Codegen writes `convex/crons.ts` from `confect/crons.ts`.
- Do not pass a time zone to `Cron.parseUnsafe`. A time zone other than UTC throws when the module loads. Convex runs every cron in UTC.
- Codegen writes the files in `convex/`, except `convex.config.ts` and `tsconfig.json`. You own those two files. Edit `convex/convex.config.ts` yourself when a component changes.
