# js knowledge

Covers 8 topics: ref-sp-js-javascript, ref-sp-js-next, ref-sp-js-next-template, ref-sp-js-react, ref-sp-js-typescript, ref-spst-js-elysium, ref-spst-js-next, ref-spst-js-styling.
---

# ref-sp-js-javascript

_When this applies: Portable JavaScript guidance for scripts and browser code with JSDoc-based typing. Use when: writing plain JavaScript, adding JSDoc, or keeping JavaScript maintainable without TypeScript._

# JavaScript

## Purpose

Provide portable defaults for maintainable JavaScript when full TypeScript is not the right tool, especially for scripts and browser-side code that still benefit from structure and JSDoc.

## When to use this skill

- Writing plain JavaScript for scripts, browser code, or small tools.
- Adding or improving JSDoc types.
- Refactoring dynamic JavaScript into clearer, more explicit code.
- Reviewing whether a JavaScript module should stay JS or move to TypeScript.

## Scope Boundaries

- Use this skill for portable JavaScript structure and JSDoc guidance.
- Use `ref-sp-js-react` when the main question is about React component structure, hooks, or React-specific dependency choices, whether the file is JavaScript or TypeScript.
- Use `ref-sp-js-next` when Next.js framework concerns dominate the design.
- Use `ref-sp-js-typescript` when the main question is about strict type-system design rather than JSDoc-backed JavaScript.
- Use `ref-sp-dev-coding-patterns` for language-agnostic naming, comments, CLI ergonomics, and testing defaults.
- Use `ref-sp-dev-projects-architecture` for portable feature-boundary or shared-utility decisions.
- Use `ref-sp-js-userscript` or `ref-sp-js-web-standalone-template` when the JavaScript lives inside a userscript or standalone browser app and those constraints dominate the design.

## Defaults

- Prefer modern ESM syntax.
- Prefer TypeScript for modern Node and Deno code when the runtime can execute `.ts` or `.mts` directly and the files are not shipped as package runtime from `node_modules`.
- Modern Node can execute TypeScript directly through built-in type stripping, so do not choose JavaScript merely to avoid `tsc`, `ts-node`, or a build runner.
- For no-build npm packages or Git-installed CLIs that execute from `node_modules`, use `.mjs` JavaScript with JSDoc and active `checkJs` type checking instead of shipping `.ts` or `.mts` runtime files.
- Use plain JavaScript intentionally for browser-delivered code, JSDoc-first modules, or repos that have already chosen JS as the local default.
- When code intentionally stays JavaScript, prefer `.js` for most modules and `.mjs` for executable ESM scripts or entrypoints where the runtime boundary should be unambiguous.
- Prefer JSDoc on exported helpers, shared objects, and non-obvious callbacks.
- Prefer `/** @type {const} */` on fixed literal maps and tuples when the exact keys or values matter; do not widen them to broad `Record<string, ...>` or `string[]` annotations just to make indexing easier.
- Prefer const-backed source-of-truth objects for closed sets of labels, states, or variants, and derive key or value unions from those literals in JSDoc-aware tooling instead of maintaining a parallel hand-written union.
- Prefer named constants and helpers over repeated inline logic.
- Prefer `const` arrow functions for JavaScript helpers, callbacks, and script-local functions.
- Prefer inferred return types for local helpers when JSDoc-aware tooling can infer them cleanly; write `@returns` only when the return value is part of an API contract, exported callback contract, or otherwise ambiguous.
- Prefer explicit input validation at I/O boundaries.
- Prefer simple data flow over mutation-heavy code.
- Prefer Yarn for dependency management and script execution in Node-based JavaScript projects unless the repo is intentionally Deno-owned.

## Task Framing

| Command or action | What | Why | When | Expected outcome |
| --- | --- | --- | --- | --- |
| Add JSDoc where it earns its keep | Document exported shapes, callbacks, and shared objects without turning the file into comment soup. | JavaScript stays maintainable when the implicit contracts are surfaced selectively. | When editor help or object shapes are getting hard to follow. | The file is still plain JavaScript, but the important contracts are explicit. |
| Split browser or script responsibilities | Separate DOM access, state changes, parsing, and I/O into named helpers or modules. | JavaScript gets hard to debug quickly when everything is inline and anonymous. | When a script starts mixing too many concerns. | The code reads in layers instead of as one giant callback. |
| Choose a package-level layout | Keep feature code under a package-owned `src/` tree when the repo is multi-package or monorepo-style. | Package ownership is easy to lose when scripts, shared code, and app code sit at the same level. | When introducing a reusable JS package or tool in a monorepo. | The package has a clear root and feature slices stay local to it. |
| Split linting by runtime surface | Give scripts, browser modules, and userscripts their own ESLint file globs and language options. | One lint config rarely fits Node scripts, browser code, and userscript globals equally well. | When a repo mixes plain JS, `.mjs`, and userscript files. | The lint config matches the runtime instead of forcing false positives or broad exceptions. |

## Core Rules

### JSDoc and typing

- Before choosing JSDoc-backed JavaScript for Node scripts, check whether the repo can use direct TypeScript execution in its supported Node version.
- Use `/** @import { SomeType } from './somewhere.js' */` for imported types instead of duplicating them with local `@typedef` blocks.
- Use local `@typedef` only for shapes owned by the file or for derived const-backed types that do not exist elsewhere.
- Use `@param` where it materially improves editor tooling and readability.
- Prefer inferred return types over `@returns` by default; add `@returns` for exported API contracts, complex callbacks, non-obvious unions, or cases where tooling would infer a misleading type.
- Document object shapes and callback contracts that would otherwise be implicit.
- For fixed literal lookup objects or tuples, prefer `/** @type {const} */` so TypeScript infers the specific keys and values from the literal.
- If checked JavaScript needs a derived union from a fixed lookup object, prefer a `typeof ...[keyof typeof ...]` style typedef over a duplicated string-literal union.
- If a dynamic string needs to index a const-typed lookup object, narrow the key first with a guard or a targeted cast instead of widening the whole object to `Record<string, ...>`.
- Keep JSDoc synchronized with the code; stale type comments are worse than no comments.

Example:

```js
/** @import { ReportRow } from './report-types.js' */

export const statValueLabels = /** @type {const} */ ({
  1: 'Basso',
  2: 'Medio',
  3: 'Alto',
});

/** @typedef {keyof typeof statValueLabels} StatValue */
/** @typedef {typeof statValueLabels[keyof typeof statValueLabels]} StatValueLabel */
```

### Structure

- Break repeated or mentally heavy logic into named helpers.
- Use `const name = (...) => { ... }` as the default function shape for helpers, callbacks, and script-local functions.
- Use a function declaration only when the declaration-specific behavior matters, such as intentional hoisting, generators, or compatibility with an existing API shape.
- Keep DOM access, state transitions, rendering, and event wiring conceptually separate in browser code.
- Avoid giant anonymous functions when a named local helper would clarify intent.
- Keep the module top level side-effect-free: importing should bind names, not open connections, read the environment, or build stateful clients. Export a factory and let the caller construct, so import order stays irrelevant. Wiring at an application entry point is the intended exception; see `ref-sp-dev-coding-patterns`.

### Scripts

- Use consistent flag names and help text when a JS file acts like a CLI.
- Keep script inputs explicit rather than reaching into ambient globals unless the platform requires it.
- Validate file, network, or user-provided input before acting on it.

### Library recommendations

- Prefer `citty` for Node-facing JavaScript CLIs that need argument parsing, subcommands, help text, and a maintainable command surface.
- Prefer `consola` for Node-facing JavaScript CLI logging so success, warning, and error output stay consistent without hand-rolled terminal formatting.
- Prefer Jest for Node-facing JavaScript or TypeScript package tests when one runner should cover colocated `*.test.js` and `*.test.ts` files.
- Keep CLI-specific dependencies narrow: if a local helper plus native APIs are enough, do not add a framework just because it is popular.

### File extensions and linting

- If modern Node or Deno can run the file directly, reconsider whether it should be TypeScript before defaulting to `.js`.
- Use JavaScript for Node CLIs shipped from `node_modules` without a build only because Node refuses type stripping under `node_modules`, not because Node generally needs `tsc` or `ts-node` to execute TypeScript.
- Use `.mjs` for explicit ESM entrypoints and runnable script shims when a file intentionally stays JavaScript and the extension clarifies runtime intent.
- Keep ordinary feature modules on `.js` only when the surrounding repo or delivery target actually wants JavaScript.
- When a repo mixes browser modules, repo scripts, and userscripts, split the ESLint flat config by file globs rather than diluting one config with many exceptions.

## Example Layouts

### Plain JavaScript package in a monorepo

```text
packages/package-name/
  src/
    csv-tools/
      index.js
      parse-report.js
      parse-report.test.js
```

### Browser script with local helpers

```text
src/features/example-browser-tool/
  example-browser-tool.html
  js/
    app.js
    storage.js
```

### Mixed repo with explicit ESM scripts

```text
scripts/
  generate-catalog.mts
src/features/example-data-transform/
  normalize-results.mjs
```

## Validation

- Exported helpers and shared objects have useful JSDoc where needed.
- Repeated or complex logic has been named and isolated.
- Fixed literal maps stay const-typed and do not duplicate the same closed set in a separate hand-written union unless the toolchain truly requires it.
- Browser code keeps responsibilities readable.
- Module imports stay side-effect-free; work is deferred to factories or explicit initialization rather than running at import time.
- Script inputs and outputs are explicit and predictable.
- Functions follow the local const-arrow default unless a declaration-specific behavior is intentionally needed.
- Node and Deno files are still on JavaScript only when that choice is deliberate rather than inertia.
- Node-based package installs and CLI invocations stay on Yarn unless the repo is intentionally Deno-owned.

## References

- MDN JavaScript Guide: <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide>
- MDN JavaScript Modules: <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules>
- TypeScript JSDoc Reference: <https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html>
- Review `./evals/evals.json` when validating output quality for JS structure, JSDoc, or mixed-runtime config guidance.

## Checklist

## Review Checklist

- Shared helpers and non-trivial objects use JSDoc where it materially improves readability.
- Browser code, CLI code, and Node scripts each keep their responsibilities explicit.
- Node scripts stay JavaScript only for a positive reason; modern Node direct TypeScript execution means avoiding `tsc` or `ts-node` is not by itself enough.
- In multi-package repos, JavaScript code stays under the owning package's `src/` tree instead of drifting into shared buckets by default.
- `.mjs` is used deliberately for explicit ESM entrypoints rather than sprinkled arbitrarily across ordinary modules.
- Helpers, callbacks, and script-local functions use `const` arrow functions by default, with function declarations kept only for intentional declaration behavior.
- ESLint config slices follow the real runtime surfaces instead of forcing browser, Node, and userscript code through one loose ruleset.
- DOM queries, selectors, and injected identifiers are named rather than repeated inline.
- External input is validated before business logic depends on it.
- The code stays readable without importing TypeScript-only habits that do not fit plain JavaScript.

## Config templates

## Config Templates

Use this file when the task needs concrete linting or mixed-extension examples rather than only structural guidance.

### When to load this file

- The user asks for an ESLint flat-config example.
- The repo mixes browser modules, Node-run scripts, and userscripts.
- The task is about `.js` versus `.mjs` placement or how lint globs should follow those extensions.

### Template provided

- `./assets/web-pages-eslint.config.template.mjs` — copied from the `web-pages` repository root and split into distinct config slices for browser modules, repo scripts, JavaScript userscripts, and TypeScript userscripts.

### Adaptation rules

- Keep separate file globs for browser modules, Node-run scripts, and userscripts when their globals or parser needs differ.
- Pair the script slice with `.mts` or `.ts` globs only if the repo actually executes or type-checks those script extensions.
- If the repo has only JavaScript userscripts, remove the TypeScript userscript slice instead of leaving dead config.
- Pair this template with `ref-sp-js-typescript` when the same repo also needs split `tsconfig` projects.

---

# ref-sp-js-next

_When this applies: Portable Next.js guidance for App Router structure, client and server boundaries, framework integrations, and Next-friendly library choices. Use when: creating or reviewing Next routes and layouts, deciding where 'use client' belongs, configuring Next.js, or choosing framework-specific integrations like next-intl._

# Next.js

## Purpose

Provide portable defaults for maintainable Next.js apps, especially around App Router structure, client and server boundaries, and framework-specific integrations that behave differently from plain React.

## When to use this skill

- Creating or reviewing App Router routes, layouts, and metadata.
- Deciding whether a component or feature belongs on the server or client side.
- Configuring Next.js or framework-sensitive integrations.
- Choosing Next-specific libraries such as internationalization solutions.

## Scope Boundaries

- Use this skill for Next.js framework rules, routing structure, rendering boundaries, and framework-specific integrations.
- Use `ref-sp-js-react` for general React component structure, hooks, local UI state, and React dependency choices that are not specific to Next.js.
- Use `ref-sp-js-next-template` when the user is planning or reviewing a whole React and Next app rather than one framework concern.
- Use `ref-sp-js-typescript` when the question is primarily about strict type modeling or TypeScript configuration.
- Use `ref-sp-js-web-standalone-template` when the requirement is a browser-only app that should stay framework-free and no-build by default.

## Defaults

- Prefer App Router for new Next.js work.
- Prefer Server Components by default and add `'use client'` only where interactivity or browser-only APIs require it.
- Prefer thin route files that delegate reusable UI and logic into feature code.
- Prefer strict TypeScript for Next apps unless the repo already made a different deliberate choice.
- Prefer Yarn for dependency management and script execution in Node-based Next projects.
- Prefer official framework integrations when a library needs special Next.js wiring.
- Prefer `next-intl` when a Next app genuinely needs internationalization.

## Task Framing

| Command or action | What | Why | When | Expected outcome |
| --- | --- | --- | --- | --- |
| Choose the rendering boundary | Decide whether the route, layout, or component stays on the server or crosses into `'use client'`. | Next.js stays simpler when client code is introduced deliberately instead of spreading everywhere. | When adding interactivity, browser APIs, or async data. | Server and client responsibilities are explicit. |
| Keep route files thin | Move reusable UI and feature logic out of route entry points. | Route files become hard to maintain when they own layout, data, and component orchestration at once. | When a route or layout starts growing. | App Router files stay readable and framework-focused. |
| Choose framework integrations | Use the official integration path for UI or i18n libraries that need framework wiring. | Next-specific integrations often fail in subtle ways when treated like plain React packages. | When adding MUI, i18n, or other framework-aware tooling. | The integration matches documented Next behavior. |

## Core Rules

### Routes and layouts

- Keep route entry points focused on routing, metadata, and high-level composition.
- Prefer colocating feature components outside the route file once the UI grows beyond a small route shell.
- Keep layout and page composition readable from top to bottom.

### Client and server boundaries

- Default to Server Components when the code does not need browser state, DOM APIs, or event handlers.
- Add `'use client'` at the narrowest practical boundary.
- Do not move entire route trees to the client just because one leaf needs interactivity.
- Do not construct request-scoped state at module scope: a module-level `new QueryClient()` (or a similar client) is shared across requests on the server and can leak data between users. Create such clients per request inside a client component with `useState(() => new QueryClient())` instead. See `ref-sp-dev-coding-patterns` for the general import-side-effect rule.

### Integrations and libraries

- If the app uses MUI, prefer the official MUI and Next.js integration path instead of ad hoc SSR wiring.
- Prefer `next-intl` when localization is required in App Router projects.
- Keep package manager usage consistent; do not mix Yarn with another Node package manager without a repo-wide reason.

## Validation

- Route and layout files stay thin and framework-focused.
- Client boundaries are narrow and justified.
- Framework-specific libraries use the official Next integration path.
- Internationalization is added only when the app actually needs it.

## References

- Next.js Documentation: <https://nextjs.org/docs>
- Next.js App Router Documentation: <https://nextjs.org/docs/app>
- MUI Next.js Integration: <https://mui.com/material-ui/integrations/nextjs/>
- next-intl App Router Guide: <https://next-intl.dev/docs/getting-started/app-router>
- Review `./evals/evals.json` when validating output quality for Next framework guidance.

## Checklist

## Review Checklist

- Route and layout files stay thin instead of accumulating most of the app's logic.
- `'use client'` appears only where browser interactivity or browser-only APIs genuinely require it.
- Reusable UI and feature logic live outside route entry files once the route shell grows.
- Next-specific integrations use their documented framework setup instead of ad hoc workarounds.
- Internationalization is introduced only when the app actually needs it, with `next-intl` as the default Next-specific choice.
- Yarn is the explicit package manager for Node-based Next.js work unless the repo already standardized differently.

---

# ref-sp-js-next-template

_When this applies: App-level guidance for building a full React and Next.js application. Use when: scaffolding or reviewing a whole React/Next app, choosing the baseline stack and package manager, deciding app-level structure, or comparing a full app against a simpler standalone browser tool._

# React + Next App

## Purpose

Provide app-level defaults for planning and reviewing a whole React and Next.js application without duplicating the lower-level framework guidance that belongs in `ref-sp-js-react`, `ref-sp-js-next`, and `ref-sp-js-typescript`.

## When to use this skill

- Scaffolding a new React and Next.js app.
- Reviewing whether an existing React and Next app has a coherent baseline stack and folder structure.
- Deciding whether a requirement deserves a full app or can stay a simpler standalone browser tool.
- Choosing app-level defaults such as package manager, TypeScript baseline, and top-level structure.

## Scope Boundaries

- Use this skill for whole-app setup, baseline stack decisions, and top-level app structure.
- Use `ref-sp-js-react` for component internals, hooks, local UI state, and general React dependency choices.
- Use `ref-sp-js-next` for App Router structure, client and server boundaries, framework integrations, and Next-specific library choices.
- Use `ref-sp-js-typescript` for strict type modeling, runtime boundaries, and `tsconfig` design.
- Use `ref-sp-js-web-standalone-template` when the requirement can stay a no-build browser app instead of becoming a full React and Next project.

## Defaults

- Prefer Next.js App Router plus strict TypeScript as the baseline for a whole React and Next app.
- Prefer Yarn for dependency management and script execution in Node-based React and Next apps.
- Keep the top-level app structure small and feature-first.
- Prefer one coherent UI system across the app instead of mixing multiple styling and component stacks.
- Decide early whether the app can stay static-friendly or genuinely needs server-dependent features.

## Task Framing

| Command or action | What | Why | When | Expected outcome |
| --- | --- | --- | --- | --- |
| Choose the app baseline | Set the package manager, framework baseline, and TypeScript level before feature work sprawls. | App-level drift becomes expensive once routes, components, and scripts are already spread across the repo. | When starting or re-baselining a React and Next app. | The app has one clear baseline stack and workflow. |
| Shape the top-level layout | Decide how routes, features, shared UI, and styles should be separated. | Whole apps get hard to navigate when route files, feature code, and shared UI all live at the same level. | When scaffolding or refactoring the app shell. | The main source tree has clear app-level ownership boundaries. |
| Compare app versus standalone | Judge whether the requirement truly needs React and Next or can stay a simpler browser app. | Full app infrastructure is expensive when the problem could stay no-build and local. | When the requested feature looks like a small tool or local utility. | The chosen app model matches the actual problem size. |

## Core Rules

### App baseline

- Make the package manager explicit and keep the Node-based workflow on Yarn consistently.
- Keep the initial app baseline minimal instead of front-loading many optional libraries.
- Let `ref-sp-js-react` and `ref-sp-js-next` own the detailed framework rules instead of copying them into this skill.

### Top-level structure

- Keep the route tree, feature code, shared UI, and app-shell code in predictable separate areas.
- Designate one composition root — the root layout and its providers — as the place to wire app-wide singletons and context providers; keep ordinary feature modules import-inert so importing them does no work. See `ref-sp-js-react` and `ref-sp-js-next` for the per-request and `use client` specifics.
- Keep app-level ownership boundaries obvious before worrying about lower-level route or component refinements.
- Avoid leaving prototypes, migration scripts, or maintenance helpers inside the main app tree.

### Decision rules

- If the requirement is a local browser app that can stay framework-free, prefer `ref-sp-js-web-standalone-template` instead of escalating to a full app.
- If the question turns into component internals, hook design, or React dependency choices, load `ref-sp-js-react`.
- If the question turns into App Router, `use client`, metadata, or Next-specific integrations, load `ref-sp-js-next`.

## Validation

- The app baseline clearly names Next.js, TypeScript, and Yarn.
- The top-level layout separates routes, feature code, shared UI, and maintenance surfaces intentionally.
- The chosen app model is justified instead of using full app infrastructure by habit.
- Lower-level React, Next.js, and TypeScript guidance is sourced from the dedicated reference skills instead of being duplicated here.

## References

- Read `ref-sp-js-react` for component, hook, and React dependency guidance.
- Read `ref-sp-js-next` for App Router, rendering-boundary, and Next-specific integration guidance.
- Read `ref-sp-js-typescript` for strict TypeScript and runtime-boundary guidance.
- Read `ref-sp-js-web-standalone-template` when the feature may not need a full React and Next app.
- Review `./evals/evals.json` when validating output quality for app-level planning guidance.

## App stack

## App Stack Baseline

### Baseline choices

- For a full React and Next app, default to `next`, `react`, `react-dom`, and `typescript`.
- Use Yarn as the default package manager and script runner for the Node-based app workflow.
- Keep framework-sensitive dependency choices delegated to the framework reference skills:
  - `ref-sp-js-react` for UI system, validation, async server state, and date-library choices.
  - `ref-sp-js-next` for App Router integrations such as `next-intl` and MUI's Next wiring.

### Structure rule

- Start with a small app shell and add folders only when they earn their keep.
- Keep route entry files thin and put reusable feature logic outside the route tree.
- Keep maintenance scripts, migrations, and scaffolding outside the main app source tree.

### Escalation rule

- If the requirement is a browser-only tool that can stay no-build, use `ref-sp-js-web-standalone-template` instead of creating a full app.
- If the requirement already needs routing, app-wide navigation, SSR or RSC boundaries, or full app deployment concerns, the full React and Next baseline is justified.

## Checklist

## Review Checklist

- The app baseline explicitly chooses Next.js, strict TypeScript, and Yarn for the Node-based workflow.
- The top-level source tree separates routes, feature code, shared UI, and maintenance surfaces intentionally.
- The app did not absorb framework or tooling complexity that a standalone browser app could have avoided.
- React component internals and dependency choices are delegated to `ref-sp-js-react` instead of duplicated here.
- Next.js rendering, routing, and integration rules are delegated to `ref-sp-js-next` instead of duplicated here.
- Package manager usage stays consistent rather than mixing multiple Node package managers in the same app.

---

# ref-sp-js-react

_When this applies: Portable React guidance for components, hooks, client-side state, and React-friendly library choices. Use when: creating or reviewing React components, choosing React libraries, deciding where UI or async state should live, or refactoring a React feature that is getting hard to read._

# React

## Purpose

Provide portable defaults for readable React code, disciplined hook boundaries, and dependency choices that solve real UI complexity without growing a stack by accident.

## When to use this skill

- Creating or refactoring React components or hooks.
- Deciding where local UI state, derived state, and async server state should live.
- Choosing React libraries for UI, validation, caching, or date handling.
- Reviewing whether a React feature is too coupled, too stateful, or carrying unnecessary dependencies.

## Scope Boundaries

- Use this skill for React component design, hook conventions, and React-focused library choices.
- Use `ref-sp-js-next` when the main question is about App Router, routing, client or server component boundaries, metadata, or other Next.js-specific integrations.
- Use `ref-sp-js-typescript` when the main question is about strict type modeling, runtime trust boundaries, or `tsconfig` design rather than React behavior.
- Use `ref-sp-js-javascript` when the code intentionally stays plain JavaScript with JSDoc and the main question is not React-specific.
- Use `ref-sp-js-next-template` when the user is planning or reviewing a whole React and Next app rather than one React feature.
- Use `ref-sp-js-web-standalone-template` for no-build browser apps that should stay framework-free by default.

## Defaults

- Keep components focused enough that the render flow is easy to scan in one pass.
- Prefer local state for local UI concerns and derive values during render instead of mirroring props into state.
- Prefer extracted custom hooks when effects, subscriptions, or async coordination repeat across components.
- Prefer one coherent UI system when a component library is justified.
- Prefer runtime validation and async caching libraries only when the feature actually crosses those boundaries.
- Prefer native `Date` and `Intl` first, and use `date-fns` only when date logic becomes genuinely complex.
- For closed component variant maps, label tables, and route or status dictionaries, prefer const literals as the source of truth and derive unions from them instead of maintaining a second parallel type.
- Prefer inferred return types for components, hooks, and local helpers; annotate returns only for exported API contracts, framework callbacks, reusable hooks with non-obvious contracts, or cases where inference would widen the type incorrectly.
- For Node-based React projects, prefer Yarn for dependency management and script execution unless the repo is intentionally Deno-owned.

## Task Framing

| Command or action | What | Why | When | Expected outcome |
| --- | --- | --- | --- | --- |
| Review component boundaries | Separate rendering, async work, and state orchestration into components or hooks that can be read in one pass. | React becomes fragile when one file owns layout, effects, mutations, and validation all at once. | When a component is growing or mixing concerns. | The feature is easier to reason about and test. |
| Choose the state owner | Decide whether state stays local, becomes derived, moves into context, or becomes query-managed. | React code stays simpler when each state value lives at the narrowest level that actually needs it. | When adding filters, forms, dialogs, or remote data. | State placement matches who reads and updates it. |
| Choose dependencies deliberately | Add a library only if it removes real complexity, with one default per problem type. | React stacks bloat quickly when every concern gets a new helper package. | When evaluating UI, validation, async caching, or date handling. | The feature uses the smallest justified dependency set. |

## Core Rules

### Component structure

- Keep render logic obvious from top to bottom.
- Split components when one file heavily mixes layout, data loading, mutations, and event choreography.
- Prefer composition and explicit props over deep wrapper stacks or implicit children contracts.
- Preserve semantic HTML even when a component library provides generic wrappers. A native element
  carries role, focusability, and keyboard behaviour that a styled `<div>` does not, and every
  capability you drop has to be rebuilt in ARIA and kept correct — see
  `ref-sp-ux-accessibility`.
- When a component relies on a closed set of variants or labels, keep the canonical list in a const object or tuple and derive the prop or state union from that value.
- In JSDoc-backed React files, import external types with `/** @import { SomeType } from './somewhere.js' */` instead of duplicating typedefs locally.

### Hooks and state

- Keep state local if one component owns it.
- Lift state only when multiple consumers truly need to coordinate on the same value.
- Use derived values during render instead of copying props or query results into extra state.
- Treat effects as synchronization with external systems, not as a fallback place for ordinary calculations.
- Prefer `useEffectEvent`, `startTransition`, and `useDeferredValue` when they clarify event or effect boundaries or keep interactions responsive.
- Do not add `useMemo` or `useCallback` by default unless profiling or an established repo convention shows that they earn their keep.
- Avoid hand-maintained variant unions that duplicate a const dictionary used for rendering badges, tabs, labels, or route metadata; derive those types from the value instead.

### Library recommendations

- Keep library additions rare and intentional.
- For component-heavy React apps that need a ready-made system, prefer MUI with `@emotion/react` and `@emotion/styled`; if that stack needs icons, prefer `@mui/icons-material`.
- Prefer `zod` for runtime schema validation at form, URL, persistence, and network boundaries.
- Prefer `@tanstack/react-query` for async server state, caching, invalidation, and background refresh when that complexity is real.
- Instantiate an app-wide client such as a TanStack `QueryClient` at the composition root, not at the top of an imported module. In a purely client-side app a single module-level instance is acceptable; when the code can also run on the server, create it inside the component with `useState(() => new QueryClient())` so it is not shared across requests. See `ref-sp-dev-coding-patterns` for the general import-side-effect rule.
- Avoid mixing styling systems casually; if an app already uses a component and theme system such as MUI, do not add Tailwind or a second styling stack without a repo-wide reason.

## Validation

- Components have a clear owner for local UI state, derived values, async state, and validation.
- Effects synchronize with external systems rather than duplicating render-time derivations.
- Library choices match concrete needs and avoid mixed-stack sprawl.
- Extracted hooks or child components improve readability instead of hiding simple logic.

## References

- React Documentation: <https://react.dev/>
- MUI Documentation: <https://mui.com/material-ui/getting-started/>
- TanStack Query Documentation: <https://tanstack.com/query/latest/docs>
- Zod Documentation: <https://zod.dev/>
- date-fns Documentation: <https://date-fns.org/>
- Review `./evals/evals.json` when validating output quality for React conventions and dependency guidance.

## Checklist

## Review Checklist

- Components and hooks have clear ownership boundaries instead of mixing rendering, effects, mutations, and validation in one place.
- State lives at the narrowest level that actually needs to read or update it.
- Effects synchronize external systems rather than recomputing ordinary derived values.
- Library additions are justified and match the default recommendations for the actual problem.
- The UI and styling stack is coherent instead of mixing overlapping systems casually.
- Query or mutation abstractions are used only when caching, invalidation, retries, or background refresh genuinely matter.
- Date helpers and validation libraries appear only where native APIs or small local helpers stop being clear.

## Library recommendations

## Library Recommendations

### Dependency rule

- Start with `react`, `react-dom`, and platform APIs.
- Add a dependency only when it deletes code, removes a recurring failure mode, or gives a capability the platform does not provide cleanly.
- Prefer one default library per concern rather than keeping multiple overlapping options open.
- For Node-based React projects, prefer Yarn for installs and scripts unless the repo is intentionally Deno-owned.

### UI and styling

- For component-heavy React apps that need a ready-made design system, default to `@mui/material` with `@emotion/react` and `@emotion/styled`.
- If the repo already has a design system or component library, stay inside that system instead of mixing multiple component stacks.
- When the stack is MUI, prefer `@mui/icons-material` for icons.
- Avoid mixing Tailwind with MUI unless the repo has already standardized on that combination.

### Validation and data boundaries

- Prefer `zod` when forms, search params, persisted data, or network payloads need runtime validation and inferred types.
- Keep schemas close to the feature that owns the trust boundary.

### Async server state

- Prefer `@tanstack/react-query` when the UI needs caching, invalidation, background refresh, optimistic updates, or request deduplication.
- Do not add it for one-off fetches that can stay inside a small feature without those lifecycle concerns.

### Dates

- Prefer native `Date` and `Intl` for basic formatting, trusted ISO parsing, and simple comparisons.
- Reach for `date-fns` only when date arithmetic, ranges, locale helpers, or formatting complexity starts to obscure the feature code.

### Non-defaults

- Avoid adding multiple overlapping state, form, or styling libraries to the same small feature.
- If the app is Next.js and the question is about i18n, routing, metadata, or framework integration, load `ref-sp-js-next`.
- If a new dependency is not one of these defaults, document why the default did not fit.

---

# ref-sp-js-typescript

_When this applies: Portable TypeScript guidance for strict typing, runtime boundaries, and maintainable scripts or apps. Use when: writing or reviewing TypeScript code, types, or configuration decisions._

# TypeScript

## Purpose

Provide portable TypeScript defaults that keep types honest, runtime boundaries explicit, and code readable under strict settings.

## When to use this skill

- Writing or refactoring TypeScript modules or scripts.
- Deciding how to model states, responses, and shared types.
- Reviewing TypeScript config and type-checking strictness.
- Handling unknown external data safely.

## Scope Boundaries

- Use this skill for strict TypeScript design, runtime boundaries, and package-level structure.
- Use `ref-sp-js-react` when the main question is about React component structure, hooks, client-side state ownership, or React-specific dependency choices.
- Use `ref-sp-js-next` when the main question is about Next.js framework structure, App Router, or Next-specific integrations.
- Use `ref-sp-js-javascript` when the code intentionally stays in plain JavaScript with JSDoc rather than full TypeScript.
- Use `ref-sp-dev-coding-patterns` for language-agnostic naming, comments, CLI ergonomics, and testing defaults.
- Use `ref-sp-dev-projects-architecture` for generic feature-boundary or shared-utility decisions that are not TypeScript-specific.

## Defaults

- Prefer strict mode and keep it strict.
- Prefer `unknown` plus narrowing over `any`.
- Prefer discriminated unions for stateful variants.
- Prefer explicit runtime validation at trust boundaries.
- Prefer inferred function return types by default when TypeScript can express the result cleanly from the implementation.
- Add explicit return annotations when the function defines a public API contract, implements an interface or overload, is recursive or generic enough to infer poorly, returns a deliberately narrow union, or crosses a framework/library boundary where the contract matters more than the implementation.
- Prefer `as const` for fixed literal maps and tuples when the exact keys or values matter; do not widen them to `Record<string, ...>` or `string[]` unless the surface is intentionally open-ended.
- Prefer const data as the source of truth for closed sets: derive key and value unions from `as const` objects or tuples instead of maintaining a parallel hand-written type that can drift.
- Prefer TypeScript over plain JavaScript in modern Node and Deno codebases because current runtimes can execute `.ts` and `.mts` directly.
- Modern Node can run TypeScript directly through built-in type stripping; do not add `ts-node`, `tsx`, or a build step just to execute ordinary Node-owned `.ts` or `.mts` scripts.
- Prefer `.ts` for ordinary TypeScript modules, colocated feature tests, and most feature code.
- Prefer `.mts` for Node ESM scripts that are executed directly by Node and need the extension to communicate module format clearly.
- Prefer `const` arrow functions for TypeScript helpers, callbacks, components, and script-local functions.
- Prefer Yarn for dependency management and script execution in Node-based TypeScript projects unless the repo is intentionally Deno-owned.

## Task Framing

| Command or action | What | Why | When | Expected outcome |
| --- | --- | --- | --- | --- |
| Model a runtime boundary | Decide where external data becomes validated domain data. | TypeScript is safest when compile-time certainty matches runtime reality. | When reading network, file, env, or parsed JSON input. | The code narrows or validates untrusted input before use. |
| Place types and features | Keep package-owned code and its local types under a readable `src/<feature>/` layout. | Deeply shared `types/` folders often grow faster than the actual features they serve. | When starting or reorganizing a package. | The feature and the types it owns are easy to find together. |
| Review strictness and readability | Check whether utility types, generics, and unions are helping or obscuring the model. | Strictness loses value when the types stop communicating intent. | When reviewing or refactoring a non-trivial TypeScript module. | Runtime safety stays strong without burying the domain in type tricks. |

## Core Rules

### Type design

- Model states and variants with unions instead of optional-property soup.
- Use utility types sparingly and only when they clarify intent.
- Keep literal lookup tables precise with `as const`, then narrow dynamic keys with `keyof typeof ...` or a guard instead of throwing away the literal information.
- When a fixed lookup object already defines the allowed states, labels, or variants, derive unions from it instead of duplicating the same domain in a separate alias or enum.
- Small helpers such as `type Keys<T extends object> = keyof T` and `type Values<T extends object> = T[Keys<T>]` are fine when they make those derived unions easier to read and reuse.
- Avoid deep type-level cleverness when a simple domain type would read better.

### Const-first modeling

- Prefer this pattern for closed maps and label sets:

```ts
export const statValueLabels = {
  1: 'Basso',
  2: 'Medio',
  3: 'Alto',
} as const;

export type Keys<T extends object> = keyof T;
export type Values<T extends object> = T[Keys<T>];

export type StatValue = Keys<typeof statValueLabels>;
export type StatValueLabel = Values<typeof statValueLabels>;
```

- Use an explicit `Record<...>` annotation only when the object is intentionally open-ended or must satisfy a broader external contract.

### Runtime boundaries

- Treat network data, filesystem data, environment variables, and parsed JSON as untrusted until validated.
- Narrow `unknown` with guards or validation helpers before use.
- Do not let compile-time confidence hide missing runtime checks.

### Direct Node execution

- Use direct Node TypeScript execution for Node-owned scripts when the supported Node version includes built-in type stripping; older Node 22 releases may need `--experimental-strip-types`.
- Keep directly executed TypeScript erasable: avoid syntax that requires transformation, such as enums, namespaces with runtime output, or parameter properties, unless the repo intentionally enables a transform path.
- Direct execution is not type checking. Keep `tsc --noEmit`, editor checks, or another explicit checker when correctness depends on TypeScript diagnostics.
- Do not ship directly executed `.ts` or `.mts` files as no-build package runtime from `node_modules`; Node rejects type stripping there, so publish JavaScript or use `.mjs` plus JSDoc/checkJs when no build exists.

### Code structure

- Keep module top level free of side effects: importing a module should bind names, not open connections, read environment, or construct stateful clients. Export a factory such as `export const createDb = () => connect(env.databaseUrl)` instead of a ready-made instance, so imports stay order-independent and tree-shaking and test isolation keep working.
- Construction at an application entry point or composition root is the intended exception; the rule targets work triggered by *importing* an ordinary module. See `ref-sp-dev-coding-patterns` for the portable rule.
- Keep types close to the feature or module that owns them.
- Extract shared types only when they are truly shared.
- Prefer readable named `const` arrow functions and objects over dense callback chains.
- Use function declarations only when the declaration form is materially useful, such as overload implementations, generators, intentional hoisting, or matching an existing framework/API convention.
- Let ordinary helper return types be inferred; annotate return types when they communicate an API boundary or prevent accidental widening.

### File and config conventions

- Use `.ts` for package code, browser-oriented modules, and most feature files.
- Use `.mts` for repo scripts or Node-run ESM entrypoints when the runtime or surrounding repo conventions rely on explicit module extensions.
- Use `.d.ts` for ambient declarations or globals that support the main code, such as userscript global definitions.
- Split `tsconfig` files by runtime surface when a repo mixes Node scripts, browser modules, and userscripts instead of forcing one project file to describe incompatible environments.
- Prefer Jest when a Node-managed package needs one test runner for colocated JavaScript and TypeScript feature tests.

## Example Layouts

### TypeScript package in a monorepo

```text
packages/package-name/
  src/
    invoice-import/
      index.ts
      parse-csv.ts
      parse-csv.test.ts
      types.ts
```

### Small feature with local validation

```text
src/
  session-state/
    index.ts
    validate-session.ts
    validate-session.test.ts
```

### Node ESM scripts with a dedicated TypeScript project

```text
scripts/
  generate-catalog.mts
  refresh-manifest.mts
tsconfig.json
```

## Gotchas

- Strict compile-time checks do not replace runtime validation for external data.
- `unknown` is only safer than `any` if the code actually narrows it before use.
- Large type-level abstractions can hide the domain model instead of clarifying it.
- Parallel literal types and literal objects drift unless one becomes the clear source of truth; prefer the const object and derive from it.
- Node's TypeScript execution strips types; it does not replace a checker or make non-erasable TypeScript syntax safe in every runtime mode.
- A side effect at module top level — a `connect(...)` or `new Client(...)` bound at import — breaks `sideEffects: false` tree-shaking and makes import order significant; defer it to an exported factory.
- No-build package runtime under `node_modules` is the important exception: use emitted JavaScript or JSDoc-backed `.mjs` there.

## Validation

- No new `any` escapes or unchecked casts were introduced without strong justification.
- External input is validated before domain logic uses it.
- Shared types are easy to locate and easy to understand.
- Module imports have no side effects; connections and stateful clients are built by exported factories, not at import time.
- Functions follow the local const-arrow default unless overloads, generators, intentional hoisting, or an API convention justify a declaration.
- The code does not stay on plain JavaScript merely to avoid `tsc`, `ts-node`, or a build step that modern Node and Deno runtimes no longer need.
- The resulting code remains readable to someone who did not write the types.

## References

- TypeScript Handbook: <https://www.typescriptlang.org/docs/>
- TSConfig Reference: <https://www.typescriptlang.org/tsconfig>

## Checklist

## Review Checklist

- External or untrusted data is validated at runtime before the rest of the code treats it as trusted.
- `any`, unchecked assertions, and wide casts are rare and justified.
- Types live close to the feature that owns them unless genuine sharing forces extraction.
- In multi-package repos, feature code stays under the owning package's `src/<feature>/` layout rather than under a global shared-types dump.
- Node-owned scripts use direct TypeScript execution when supported; `ts-node`, `tsx`, or build steps are added only for a real transform, compatibility, or packaging reason.
- Separate `tsconfig` project files exist when scripts, browser modules, and userscripts need different runtime assumptions.
- `.mts` is reserved for Node-run ESM scripts or entrypoints where the explicit module extension actually helps.
- Helpers, callbacks, components, and script-local functions use `const` arrow functions by default, with declaration forms reserved for overloads, generators, intentional hoisting, or API conventions.
- Complex unions, generics, or mapped types clarify the domain instead of obscuring it.
- The final code remains readable to an engineer who did not write the original types.
- No-build package runtime under `node_modules` emits JavaScript or uses `.mjs` plus JSDoc/checkJs instead of relying on Node type stripping.

## Config templates

## Config Templates

Use these when the task needs concrete TypeScript project configuration rather than only structural advice.

### When to load this file

- The user asks for a `tsconfig` example or starter template.
- The repo mixes Node scripts, browser modules, and userscripts and needs separate project files.
- The task is about `.ts` versus `.mts` placement or how config globs should follow those file extensions.

### Available templates

- `./assets/web-pages-tsconfig.base.template.json` — base options copied from `web-pages/tsconfig.base.json`. This is a mixed web-oriented base and includes `allowJs` plus `verbatimModuleSyntax`.
- `./assets/web-pages-tsconfig.node-scripts.template.json` — copied from `web-pages/tsconfig.json` for Node ESM scripts such as `scripts/*.mts` and `scripts/*.ts`.
- `./assets/web-pages-tsconfig.web.template.json` — copied from `web-pages/tsconfig.web.json` for browser modules such as `src/features/**/*.js` and `src/features/**/*.mjs`.
- `./assets/web-pages-tsconfig.userscripts.template.json` — copied from `web-pages/tsconfig.userscripts.json` for `*.user.js`, `*.user.ts`, and related declaration files.

### Adaptation rules

- Treat these as starting points, not drop-in defaults for every repo.
- Keep the file-extension globs aligned with the actual runtime surface. If the repo uses `.mts` for Node-run ESM scripts, include them explicitly. If it uses only `.ts`, remove the unused globs instead of keeping noise.
- If the repo does not mix Deno with the same TypeScript base settings, remove `deno.ns` or split that concern into a Deno-specific config.
- If userscripts are TypeScript-authored, keep a separate userscripts project file so browser modules and userscript globals do not leak into each other.
- Pair these templates with the ESLint flat-config template in `ref-sp-js-javascript`/` when the repo also needs linting slices for scripts, browser modules, and userscripts.

---

# ref-spst-js-elysium

_When this applies: SwiftPost Elysium UI library reference. Use when: importing components from @swiftpost/elysium, choosing between base and enhanced UI primitives, wiring providers, or using Elysium-specific slots, sx utilities, and theme helpers._

# SwiftPost Elysium

## Purpose

Document the SwiftPost-specific UI layer built in `@swiftpost/elysium`. Use this skill when the task depends on package-specific imports, wrappers, theme helpers, or conventions that do not generalize to other projects.

## When to use this skill

- Importing or using any component from `@swiftpost/elysium`.
- Choosing between base (MUI wrapper) and enhanced components.
- Wiring Elysium providers or theme objects.
- Using Elysium-specific utilities like `spreadSx`.
- Confirming package-specific import paths or component names.

## Import Strategy

**Never import from `@mui/material` directly in the app package.** App UI goes through Elysium.

| What | Import From |
|------|-------------|
| MUI base components | `@swiftpost/elysium/ui/base/{ComponentName}` |
| Enhanced components | `@swiftpost/elysium/ui/{ComponentName}` |
| Hooks | `@swiftpost/elysium/ui/{hookName}` |
| Types (`SxProps`, `InferSlotsFromSlotProps`, etc.) | `@swiftpost/elysium/ui/types` |
| Style utilities (`spreadSx`) | `@swiftpost/elysium/utils/styles/sxProps` |
| Theme provider | `@swiftpost/elysium/core/ThemeProvider` |
| Cache provider | `@swiftpost/elysium/core/AppRouterCacheProvider` |
| Theme object / font | `@swiftpost/elysium/themes/gamut` |
| Static theme options | `@swiftpost/elysium/themes/gamut/static` |
| Color tokens | `@swiftpost/elysium/colors/material` |
| MUI Icons | `@mui/icons-material` |

## Base Components

Thin re-exports of MUI components. Import from `@swiftpost/elysium/ui/base/*`.

| Component | MUI Source | Notes |
|-----------|-----------|-------|
| `Autocomplete` | `@mui/material/Autocomplete` | |
| `Box` | `@mui/material/Box` | |
| `Button` | `@mui/material/Button` | |
| `Checkbox` | `@mui/material/Checkbox` | |
| `FormControl` | `@mui/material/FormControl` | |
| `FormLabel` | `@mui/material/FormLabel` | |
| `Link` | `@mui/material/Link` | Base link only. For navigation, use the enhanced `Link`. |
| `MenuItem` | `@mui/material/MenuItem` | |
| `Modal` | `@mui/material/Modal` | |
| `Radio` | `@mui/material/Radio` | |
| `RadioGroup` | `@mui/material/RadioGroup` | |
| `Select` | `@mui/material/Select` | |
| `Stack` | `@mui/material/Stack` | |
| `Text` | `@mui/material/Typography` | Always use `Text`, never `Typography`. |
| `TextField` | `@mui/material/TextField` | |

### Text Rename

MUI `Typography` is renamed to `Text` across the package.

```tsx
import Text from '@swiftpost/elysium/ui/base/Text';
import type { TextProps } from '@swiftpost/elysium/ui/base/Text';
```

## Enhanced Components

### Link

`@swiftpost/elysium/ui/Link` integrates `next/link` with MUI styling. Use it for navigation instead of raw `next/link`.

### Image

`@swiftpost/elysium/ui/Image` wraps `next/image` and falls back to plain `<img>` for simple URL sources without dimensions.

### ContentFittedStack

`@swiftpost/elysium/ui/ContentFittedStack` provides constrained centered content inside a full-width stack container.

### StackLayout

`@swiftpost/elysium/layouts/StackLayout` is the package layout primitive for full-height stacks with optional footer slots.

## Providers and Theme

- `@swiftpost/elysium/core/AppRouterCacheProvider` wraps the MUI Emotion cache for App Router.
- `@swiftpost/elysium/core/ThemeProvider` is the client wrapper around the theme.
- `@swiftpost/elysium/themes/gamut` exports the runtime theme and font.
- `@swiftpost/elysium/themes/gamut/static` exports SSR-safe theme values.

## Elysium-Specific Styling Helpers

- Use `spreadSx` from `@swiftpost/elysium/utils/styles/sxProps` when merging `sx` values.
- Use `InferSlotsFromSlotProps` from `@swiftpost/elysium/ui/types` when defining slot-based APIs.
- Wrap reusable UI primitives in `memo` and keep `componentBaseName` stable for CSS targeting.

For SwiftPost-specific styling guidance, see the `ref-spst-js-styling` skill.

---

# ref-spst-js-next

_When this applies: SwiftPost Site Template Next.js static-export conventions and constraints. Use when: creating pages, working with routing, managing client/server boundaries, or configuring Next.js in this template._

# SwiftPost Next.js Conventions

## Purpose

Define the Next.js-specific rules and constraints for this static-export project, with an emphasis on simple boundaries and maintainable patterns that fit a static site.

## When to use this skill

- Creating new pages or routes.
- Working with `'use client'` boundaries.
- Configuring Next.js (next.config.ts, metadata, etc.).
- Deciding between server and client components.

## Scope Boundaries

- Use this skill for the constraints that are specific to this template: static-export limits, the `page.tsx` + `ClientWrapper.tsx` pattern, Elysium routing wrappers, and `staticTheme` access.
- Use `ref-sp-js-next` for portable Next.js guidance — App Router structure, general client/server boundary defaults, and framework integrations. This skill layers on top of it and does not restate it.
- Use `ref-spst-dev-site-architecture` when the question is about package boundaries or where a file belongs rather than a Next.js rule.

## Core Constraint: Static Export Only

This is a **frontend-only** Next.js project exported as a static website. The following server features are **forbidden**:

- Server Actions
- API routes (`app/api/`)
- Dynamic server rendering (`getServerSideProps`, server-only `cookies()`, `headers()`)
- Middleware (runs on server)
- ISR / revalidation

Everything must work as a fully static site (`output: 'export'` in next.config.ts).

## Page Pattern

Pages follow a thin Server Component pattern to minimize the `'use client'` boundary:

1. **`page.tsx`** — Server Component. Exports metadata and renders a Client Wrapper. No hooks, no state, no `'use client'`.
2. **`ClientWrapper.tsx`** — `'use client'` component. Contains or composes the actual interactive UI.

```tsx
// app/my-route/page.tsx
import ClientWrapper from './ClientWrapper';

export const metadata = { title: 'My Route' };

const Page = () => <ClientWrapper />;
export default Page;
```

```tsx
// app/my-route/ClientWrapper.tsx
'use client';

import Dashboard from '@/features/my-feature/components/Dashboard';

const ClientWrapper = () => <Dashboard />;
export default ClientWrapper;
```

## Routing & Navigation

- **Never use `next/link` directly.** Use `Link` from `@swiftpost/elysium/ui/Link`, which wraps `next/link` with MUI styling.
- **Never use `next/image` directly.** Use `Image` from `@swiftpost/elysium/ui/Image`.
- Pages and routes live in `packages/main/src/app/`.

## Static Theme Access

To use theme values without `'use client'`, import from `@/styles/staticTheme`:

```tsx
import { staticTheme } from '@/styles/staticTheme';

// Works in Server Components — no hooks needed
const spacing = staticTheme.spacing(2); // '1rem'
```

---

# ref-spst-js-styling

_When this applies: SwiftPost Site Template styling guidance for Elysium/MUI components, sx merging, Slots and SlotProps APIs, responsive layout, and theme-token use. Use when: building UI in packages/main or packages/elysium, creating reusable styled components, shaping slot APIs, or reviewing visual/layout styling._

# SwiftPost Styling

## Purpose

Define how styling should be authored in the SwiftPost Site Template so UI remains theme-aware, responsive, override-friendly, and consistent with the local `@swiftpost/elysium` layer. Use this skill for concrete styling decisions in this repository rather than generic CSS or MUI advice.

## When To Use This Skill

- Building or reviewing styled UI in `packages/main` or `packages/elysium`.
- Creating reusable components that expose `slots`, `slotProps`, or `sx`.
- Merging incoming `sx` with component defaults.
- Choosing whether a visual rule belongs in component `sx`, the theme, a slot prop, or global CSS.
- Reviewing responsive layout, stable component sizing, or theme token usage.

## Related Skills

- Load `ref-spst-js-elysium` when you need exact component imports, wrapper names, theme exports, or Elysium helper APIs.
- Load `ref-spst-dev-site-architecture` when the styling question also affects package boundaries, route structure, feature placement, or reusable component ownership.
- Load `ref-spst-dev-code-conventions` when the styling change also needs TypeScript, React, hook, or component-quality guidance.

## Inspect First

Before changing styling, inspect the nearest existing component and its import layer:

1. Confirm whether the file lives in `packages/main` or `packages/elysium`.
2. Check existing Elysium component imports before introducing new base primitives.
3. Check whether the component already exposes `sx`, `slotProps`, `slots`, `className`, or fixed dimensions.
4. Check whether a value already exists in `packages/main/src/styles/staticTheme.ts`, `packages/main/src/styles/theme.ts`, or `packages/elysium/themes/gamut` before adding magic numbers.
5. If changing layout, check mobile and desktop behavior, not only the viewport in front of you.

## Core Workflow

1. Keep styling local to the component that owns the visual behavior.
2. Promote repeated values into the local theme or a shared component only when repetition is real and meaningful.
3. Use Elysium wrappers instead of direct MUI imports, except for MUI icons.
4. Expose `sx` and slot props on reusable components so callers can extend styling without reaching into implementation details.
5. Merge `sx` arrays with `spreadSx`; never replace incoming styles accidentally.
6. Define responsive behavior mobile-first and give fixed-format controls stable dimensions.
7. Validate with lint/type-check, and use browser screenshots or manual viewport checks for layout-sensitive changes.

## Import Rules

In `packages/main`, app UI should go through Elysium:

| Need | Import |
| --- | --- |
| Base MUI primitive | `@swiftpost/elysium/ui/base/{ComponentName}` |
| Enhanced Elysium component | `@swiftpost/elysium/ui/{ComponentName}` |
| Elysium layout primitive | `@swiftpost/elysium/layouts/{ComponentName}` |
| `SxProps`, `Theme`, `InferSlotsFromSlotProps` | `@swiftpost/elysium/ui/types` |
| `spreadSx` | `@swiftpost/elysium/utils/styles/sxProps` |
| Static theme values | `@/styles/staticTheme` |
| MUI icons | `@mui/icons-material` |

Do not import app UI primitives directly from `@mui/material`. If the Elysium layer is missing a wrapper the app genuinely needs, add or expose it in `packages/elysium` first.

## `sx` Rules

- Accept `sx?: SxProps` on reusable visual components unless there is a concrete reason not to.
- Treat incoming `sx` as additive caller intent.
- Use `spreadSx(sx)` when composing arrays because `SxProps` may already be an array.
- Put component defaults first and caller overrides last when the caller should be able to override them.
- Keep `sx` objects readable; extract a named constant only when it removes noise or avoids duplication.
- Prefer theme tokens, responsive objects, and component props over ad hoc CSS strings.

Example default-plus-override merge:

```tsx
<Stack
  sx={[
    { width: '100%', minHeight: 0 },
    ...spreadSx(slotProps.root.sx),
    ...spreadSx(sx),
  ]}
>
  {children}
</Stack>
```

## Slots And SlotProps

Use Slots and SlotProps for reusable components that need overridable internal elements. Do not add slots to one-off page sections where ordinary composition is clearer.

Default pattern:

```tsx
import { memo } from 'react';
import Box from '@swiftpost/elysium/ui/base/Box';
import Stack from '@swiftpost/elysium/ui/base/Stack';
import type { InferSlotsFromSlotProps, SxProps } from '@swiftpost/elysium/ui/types';
import { spreadSx } from '@swiftpost/elysium/utils/styles/sxProps';

const componentBaseName = 'feature-panel';

interface SlotProps {
  root: {
    children: React.ReactNode;
    sx?: SxProps;
  };
  header?: {
    children: React.ReactNode;
    sx?: SxProps;
  };
}

interface Props {
  slots?: Partial<InferSlotsFromSlotProps<SlotProps>>;
  slotProps: SlotProps;
  sx?: SxProps;
}

const FeaturePanel: React.FC<Props> = ({ slots, slotProps, sx }) => {
  const Root = slots?.root ?? Stack;
  const Header = slots?.header ?? Box;

  return (
    <Root
      className={componentBaseName}
      sx={[{ width: '100%' }, ...spreadSx(slotProps.root.sx), ...spreadSx(sx)]}
    >
      {slotProps.header != null && (
        <Header
          className={`${componentBaseName}-header`}
          sx={[{ width: '100%' }, ...spreadSx(slotProps.header.sx)]}
        >
          {slotProps.header.children}
        </Header>
      )}
      {slotProps.root.children}
    </Root>
  );
};

export type FeaturePanelProps = Props;
export default memo(FeaturePanel);
```

Slot rules:

- Name slots after their role in the component, not after the current HTML tag.
- Use `Partial<InferSlotsFromSlotProps<SlotProps>>` for optional slot overrides.
- Keep `slotProps` typed and explicit; avoid loose records of arbitrary props.
- Use a stable kebab-case `componentBaseName` for class names when internal parts need CSS targeting.
- Wrap reusable components in `memo` unless local state, refs, or behavior make that inappropriate.
- Do not expose slots only to work around a poor component boundary. Split or compose the component first.

## Responsive Layout

- Write mobile-first base styles and override at larger breakpoints.
- Prefer MUI responsive objects for simple breakpoint changes.
- Use stable dimensions for controls, grids, tiles, boards, toolbars, and media regions so labels, hover states, and loading text do not shift layout.
- Use `minWidth: 0` on flex/grid children that contain text and need to shrink.
- Use `overflow`, wrapping, or smaller local typography intentionally when content can be long.
- Do not scale font sizes with viewport width. Use component-appropriate typography variants and breakpoint-specific values when needed.

Example:

```tsx
sx={{
  display: 'grid',
  gridTemplateColumns: { xs: '1fr', md: 'minmax(0, 1fr) auto' },
  gap: { xs: 2, md: 3 },
  minWidth: 0,
}}
```

## Theme And Tokens

- Use `staticTheme` from `@/styles/staticTheme` for static app values that need to be available outside client-only theme callbacks.
- Use Elysium theme exports from `@swiftpost/elysium/themes/gamut` or `@swiftpost/elysium/themes/gamut/static` when working inside the Elysium package.
- Use spacing, palette, typography, and breakpoint tokens rather than repeating raw values.
- Keep palette changes coordinated with `packages/main/src/styles/theme.ts` and `packages/elysium/themes/gamut` instead of scattering color literals.
- Add theme tokens only when they represent a reusable product decision, not a single local spacing tweak.

## Global CSS

Use `packages/main/src/app/globals.css` for true global behavior such as resets, base document sizing, or framework-level defaults. Do not put component-specific styling there when `sx`, theme components, or a component-local wrapper can own it.

Global CSS is appropriate for:

- `html`, `body`, and root layout sizing.
- Font smoothing or base rendering behavior.
- Intentional framework/global resets.

Global CSS is not appropriate for:

- Styling a one-off page section.
- Reaching into an Elysium component because it lacks a slot or prop.
- Adding broad selectors that can surprise unrelated routes.

## Review Checklist

- Imports go through Elysium in app code.
- Incoming `sx` and slot `sx` are merged, not overwritten.
- Slot names describe roles and class names use a stable `componentBaseName`.
- Responsive behavior is mobile-first and text cannot overlap adjacent UI.
- Fixed-format UI has stable dimensions.
- Theme values are reused where they already exist.
- Global CSS changes are truly global.
- The change has been validated with `yarn lint:ci && yarn typecheck:ci` when behavior or TypeScript surfaces changed.
