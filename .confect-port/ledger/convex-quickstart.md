# Ledger: convex-quickstart

Files: SKILL.md, agents/openai.yaml, assets/icon.svg. No references/ directory.
Code samples: none. The skill has rule text only.

## SKILL.md

| Source | Action | Reason |
| --- | --- | --- |
| L1-4 frontmatter name | update | Name kept as `convex-quickstart`. The manifest script requires the name to match the directory. Other skills in the fork still use a `confect-*` name with a `convex-*` directory. Open question 2. |
| L3 description | update | Now triggers on Confect work and on adding Confect to a frontend. |
| L6 GENERATED comment | keep | Unchanged upstream line, keeps sync diffs small. |
| (new) adapted comment | add | States that the file is adapted to Confect. |
| L8 title | update | "Confect template". |
| L10 intro | keep + add | Upstream text kept. Added "Then add Confect on top of it." |
| (new) pointer paragraph | add | Points to `../confect/references/confect-patterns.md` and to the project's `convex/_generated/ai/guidelines.md` when it exists. `skills/confect/references/guidelines.md` does not exist yet. |
| L14 recipe step | keep | The recipe makes a plain Convex project. Added a sentence that says so. |
| (new) steps 2-4 | add | Install Confect packages and peers (`@effect/platform-node`, `convex` 1.45+, `@confect/test` and `convex-test` as dev, Node 24+). Move the template schema and functions into `confect/`. Update frontend `api.*` calls. Run `confect codegen`. Add `confect dev` next to the recipe's backend. |
| L15 open dev URL | keep | Renumbered (step 5). Now says "the recipe" prints the URL. |
| L16 confirm plan | keep | Renumbered (step 6). |
| L20 never re-run recipe | keep | Platform only. |
| L21 delegate to convex-expert | update | Now `confect-expert` for `confect/` and plain Convex code. |
| L22 no Postgres/Redis/Express | keep | Platform only. |
| L23 no hosting or auth | keep | Platform only. |
| L24 DEGRADATION RULE | update | Code goes under `confect/`. `convex/` is generated output. Platform fact kept: functions only run from `convex/`. Added the do-not-edit-generated rule and the owned files. Added a first step: create `convex/` and install packages. Codegen fails with `ConvexDirectoryNotFoundError` without `convex/`. Zero-docs rule kept. |
| L25 unbounded collect | update | Replaced with `reader.table().index().paginate/take`. Rule kept. Added the `DatabaseReader` binding and `publicPaginatedQuery`/`internalPaginatedQuery`. |
| L25 index not filter | update | Confect has no `.filter()`. Gap row gives `.stream()` with `Stream.filter` as the escape hatch. Added: `.paginate` takes an optional filter callback. |
| L25 withIndex only eq/gt/gte/lt/lte, no range | update | Same facts for the `.index(name, (q) => ...)` callback. |
| L25 imports from ./_generated/server and api | remove + add | Raw Convex imports are replaced by `FunctionSpec`, `GroupSpec`, `FunctionImpl`, `GroupImpl`, services and `refs`. The Confect form is added in place. |
| L25 never import convex/server | update | Confect code never uses the raw builders. `convex/server` allowed only in plain functions (escape hatch E12). |
| (new) spec must not value-import @confect/server | add | Patterns, Project layout rule. |
| L25 v.literal | update | `Schema.Literal` and `Schema.Literals`. Added `Schema.Finite` advice. |
| L25 "use node" | update | Node group: `GroupSpec.makeNode()`, `publicNodeAction`, `internalNodeAction` (E11). Same platform rule kept. Added: codegen writes `"use node"`. Do not write it by hand. |
| L25 Node builtins and Web Crypto | keep | Platform only. "http.ts" changed to "HTTP route handlers". |
| L26 reserved JS words | update | Applies to the function `name` in the spec. Reason changed. Confect rejects reserved JS words, `schema`, `http` and `crons` when it builds the spec (`validateConfectFunctionIdentifier`). Table file names follow `validateConfectTableIdentifier`. Codegen fails with `InvalidTableFilenameError`. Index `_` rule kept (`IndexNameReserved` at push). Open question 1 is closed. |
| L26 `_` table and index names | update | Same rule. Added that the file name is the table name. |
| L27 httpRouter no :param | update | Kept for plain `http.route`. Added the Effect `HttpApiEndpoint` path param form (`"/users/:id"` with `params`). `:id` form verified in the effect 4.0.0 source (HttpApiClient.ts example). |
| L27 httpAction wrapper | update | `httpActionGeneric` for plain routes (E13). Effect handlers need no wrapper. |
| L28 runQuery needs a function reference | update | Runner services plus generated refs. Same rule kept. Added that refs carry schemas. The plain Convex same-file annotation rule is now stated in the text. |
| L29 SELF-VERIFY | update | Added `npx confect codegen` before `tsc`. Added the finalize note. Anonymous and signed-in rule kept word for word. |

## agents/openai.yaml

| Source | Action | Reason |
| --- | --- | --- |
| display_name | update | "Confect Quickstart". |
| short_description | update | Confect wording. |
| icon_small, icon_large, brand_color | keep | No Convex content. |
| default_prompt | update | Confect wording. Keeps the new or existing app decision. |
| policy | keep | Unchanged. |

## assets/icon.svg

| Source | Action | Reason |
| --- | --- | --- |
| whole file | keep | Generic icon. Platform only. |

Counts for SKILL.md: kept 8, updated 14, added 4, removed 1 (L25 raw imports, replaced).

Open questions:

1. Closed. The reserved-name reason now comes from the Confect source.
2. Every adapted skill has a `confect-*` frontmatter name and a `convex-*` directory. The manifest script throws on this. This skill now matches. The other skills need the same fix, or a directory rename.
3. Many skills point to `skills/confect/references/guidelines.md`. The file does not exist yet. This skill no longer depends on it.
