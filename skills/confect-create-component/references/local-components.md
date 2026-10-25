# Local Convex Components in a Confect App

Read this file when the component should live inside the current app and does
not need to be published as an npm package.

## When to Choose This

- The user wants the simplest path
- The component only needs to work in this repo
- The goal is extracting app logic into a cleaner boundary

## Default Layout

Use this structure unless the repo already has a clear alternative pattern:

```text
convex/
  convex.config.ts
  components/
    <name>/
      convex.config.ts
      schema.ts
      <feature>.ts
```

## Workflow Notes

- Define the component with `defineComponent("<name>")`
- Write the component in plain Convex. Confect does not build component code.
- Install it from the app with `defineApp()` and `app.use(...)`. You own
  `convex/convex.config.ts`. Confect does not change it.
- Keep auth, env access, public API wrappers, and HTTP route mounting in the app
- Let the component own isolated tables and reusable backend workflows
- Add app wrappers if clients need to call into the component. Write each
  wrapper as a Confect spec and impl (`confect/<group>.spec.ts` and
  `confect/<group>.impl.ts`). Call the component with the raw ctx
  (`yield* MutationCtx`) inside `Effect.promise`. Import `components` from
  `confect/_generated/components`.
- Run `npx convex codegen --component-dir convex/components/<name>` first.
  Then run `npx confect codegen`. Confect reads the component's
  `_generated/component.js` to type the `components` registry.
- Do not put the component directory in `confect/`. Confect treats files in
  `confect/` as tables, specs, and impls.

## Checklist

- [ ] Component is inside `convex/components/<name>/`
- [ ] App installs it with `app.use(...)`
- [ ] Component owns only its own tables
- [ ] App wrappers handle client-facing calls when needed
- [ ] App wrappers are Confect specs and impls, not plain Convex functions
- [ ] `npx confect codegen` and `npx tsc --noEmit` pass
