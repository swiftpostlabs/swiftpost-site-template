# dev knowledge

Covers 6 topics: ref-sp-dev-coding-patterns, ref-sp-dev-projects-architecture, ref-spst-dev-code-conventions, ref-spst-dev-config-package, ref-spst-dev-main-package, ref-spst-dev-site-architecture.
---

# ref-sp-dev-coding-patterns

_When this applies: Portable coding defaults for readable, strongly-typed, well-tested code across languages and CLIs. Use when: choosing naming, typing, comments, branching structure, CLI ergonomics, or testing defaults in a new or existing codebase._

# Coding Patterns

## Purpose

Provide portable defaults for writing code that is explicit, intention-revealing, easy to test, and easy to maintain across languages.

## When to use this skill

- Choosing general coding defaults for a new feature or repository.
- Refactoring code that is hard to read, weakly typed, or over-coupled.
- Deciding when comments help and when they are noise.
- Designing a CLI or automation surface.
- Reviewing whether tests actually cover risky behavior.

## Scope Boundaries

- Use this skill for language-agnostic defaults such as naming, comments, branching, CLI ergonomics, and testing posture.
- Use `ref-sp-dev-projects-architecture` for folder layout, feature boundaries, and shared-utility decisions.
- Use `ref-sp-py-python`, `ref-sp-js-javascript`, or `ref-sp-js-typescript` for syntax- and runtime-specific guidance.
- Use a repo-local conventions skill (in this repo, `ref-sp-dev-repo-conventions`) when the question is about a specific repository's exact paths or commands.

## Defaults

- Prefer strict typing when the language supports it.
- Prefer explicit boundaries over clever inference at I/O edges.
- Prefer names that scream intent over vague verbs.
- Prefer early returns over deep nesting.
- Prefer small focused functions over multi-purpose handlers.
- Prefer modules whose import has no side effects; defer real work to functions, factories, or explicit initialization.
- Prefer safe CLI defaults and discoverable help.
- Prefer focused tests over broad integration-style guesswork.

## Task Framing

| Command or action | What | Why | When | Expected outcome |
| --- | --- | --- | --- | --- |
| Pick naming and data-boundary defaults | Choose intention-revealing names and explicit validation or narrowing points. | Weak names and fuzzy I/O boundaries make every later refactor harder. | When starting a new feature or cleaning up unclear code. | Call sites, helpers, and data shapes read clearly without private context. |
| Shape a CLI surface | Choose subcommands, flags, help text, and safe defaults deliberately. | CLI ergonomics become sticky quickly once other people start using them. | When creating or revising a command-line workflow. | The command is discoverable, predictable, and safe to run. |
| Review comments and tests | Decide which constraints deserve comments and which branches deserve tests. | Documentation and tests are expensive if they are broad but shallow. | When finishing a risky change or reviewing a refactor. | Comments explain real constraints and tests cover the most fragile behavior. |

## Core Rules

### Typing and data boundaries

- Type public APIs, exported helpers, and shared data structures clearly.
- Treat external input as untrusted until it is narrowed or validated.
- Prefer type guards, validation helpers, discriminated unions, or tagged states over unchecked casts.
- Avoid `any`, untyped containers, and stringly-typed state when the language gives you better options.

### Naming and structure

- Use names that describe the business meaning or effect of the code.
- Prefer names like `parseInvoiceCsv`, `loadUserProfile`, or `renderDashboard`.
- Avoid names like `handleData`, `doThing`, `misc`, or `helpers` when a narrower name exists.
- Split code by responsibility before it becomes hard to name mentally.

### Control flow

- Use early exits to remove nesting and make the happy path obvious.
- Keep conditions local and explicit rather than encoding them through side effects.
- If a branch exists because of a business rule, name that rule in code or in a short comment.

### Module initialization and import side effects

- Treat importing a module as binding names only: it must not open connections, read files or environment, call the network, run expensive work, or instantiate stateful clients, pools, or singletons.
- Keep modules order-independent. If changing import order changes behavior, something is doing work at import time that should be deferred.
- Prefer a factory function, a lazy accessor, or caller-injected dependencies over a ready-made instance built at module scope.
- Pure constants and frozen lookup data at module scope are fine; the rule targets work and side effects, not values.
- Distinguish an importable module from an application entry point or composition root. Wiring things up at an entry point — an app root, a `main()`, a provider setup — is expected, and some frameworks recommend it. The rule is that *importing* a module must not trigger the work, not that construction can never happen at module scope.
- Allow a module-scope exception only as a deliberate, documented decision with a stated reason, never as something that happens by accident.
- Give tests more latitude here because their modules are small and short-lived, but still avoid hidden import-order coupling.

### Comments

- Comment why, constraints, invariants, or business context.
- Do not comment what the code already says clearly.
- Add comments when a rule is externally imposed, legally required, surprising, or easy to break during refactors.
- Treat a comment's assertion as a claim, not decoration: verify a stated invariant or safety condition (such as "safe because X") against the code before writing it as fact.

### CLI ergonomics

- Prefer verb-oriented subcommands for multi-action CLIs.
- Provide long flags and short aliases for the flags used most often.
- Use predictable pairs like `--from` and `--to` for source and destination.
- Default to the current working directory when that behavior is unsurprising.
- Add `--dry-run` for destructive or high-impact actions.
- Make `--help` concrete, with real examples and safe defaults.

### Testing

- Add focused tests for non-trivial logic, error handling, and boundary conditions.
- Cover the branch that is easiest to break, not just the happy path.
- Keep tests readable enough that they explain the intended behavior.
- Prefer a few well-named fixtures/builders over huge shared setup blocks.

## Validation

- Public functions and shared structures are typed clearly.
- Names reveal intent without needing surrounding explanation.
- Comments explain non-obvious constraints rather than narrating syntax.
- Importing a module does no I/O or heavy construction; such work is deferred to factories, lazy accessors, or an explicit entry point.
- CLI flags are consistent, discoverable, and safe.
- Tests cover risky logic and failure modes, not only success paths.

## References

## Checklist

## Review Checklist

- Public interfaces are typed clearly and avoid ambiguous data shapes.
- Names explain purpose without depending on surrounding comments.
- Branching stays readable through early returns, small helpers, or named conditions.
- Importing a module has no side effects; work is deferred to factories, lazy accessors, or an explicit entry point.
- Comments explain constraints, tradeoffs, or surprising behavior rather than narrating syntax.
- Validation and tests cover risky inputs, unhappy paths, or mutation-heavy logic.

---

# ref-sp-dev-projects-architecture

_When this applies: Portable architecture guidance for feature folders, boundaries, shared utilities, and separating product code from maintenance scripts. Use when: deciding where code should live, splitting features, or reviewing repository structure._

# Projects Architecture

## Purpose

Provide portable repository and feature-structure defaults that keep codebases modular, discoverable, and resistant to accidental coupling.

## When to use this skill

- Designing the folder layout for a new feature or tool.
- Deciding whether code belongs in product source or in maintenance scripts.
- Reviewing whether a feature has grown beyond one file or one folder.
- Choosing when shared utilities are justified.
- Refactoring a repository that has started to blur boundaries.

## Scope Boundaries

- Use this skill for portable structure decisions such as feature boundaries, shared-utility thresholds, and product-versus-maintenance separation.
- Use a repo-local conventions skill (in this repo, `ref-sp-dev-repo-conventions`) when the question is about a specific repository's exact top-level folders, `pyproject.toml`, or agent wiring.
- Use `ref-sp-py-python`, `ref-sp-js-javascript`, or `ref-sp-js-typescript` when the question is about language-specific folder shapes.
- Use `ref-sp-js-web-standalone-template` when the structure question is specific to a browser-only local app or mini-tool.

## Defaults

- Prefer feature-first organization over type-based dumping grounds.
- Keep product code under the main source tree.
- Keep maintenance and repo automation separate from product features.
- Keep tests near the code they explain when the project layout allows it.
- Add a short `README.md` at the root of each real feature folder so the feature's purpose and entrypoints are obvious without reading code first.
- Extract shared utilities only after real reuse appears.
- In mixed Python and Node repositories, keep Python package features under `src/<package>/<feature>/` and keep Node or TypeScript package features in the matching feature slice instead of inventing a parallel top-level tree.
- In mixed Python and Node repositories, use `scripts/` for thin runtime entrypoint shims, dev automation, migrations, or one-off maintenance, not for the owning implementation of installed commands.

## Task Framing

| Command or action | What | Why | When | Expected outcome |
| --- | --- | --- | --- | --- |
| Define feature boundaries | Decide what belongs inside one feature slice and what should stay outside it. | Clear ownership reduces coupling and makes refactors cheaper. | When creating a new feature or splitting a large one. | The owning folder is obvious and internal details stay internal. |
| Promote product code out of maintenance paths | Move shipped or user-facing behavior under the main source tree instead of leaving it in `scripts/`. | Prototype placement often lingers long after the code becomes part of the product. | When a script becomes a real feature or installed command. | Product code is discoverable and tested like the rest of the source tree. |
| Decide whether to share a utility | Judge whether repeated logic is stable enough to extract or still cheap to duplicate locally. | Premature sharing creates invisible dependencies and vague utility buckets. | When two or more features start to reuse similar helpers. | Shared code exists for real reuse and has a clear home. |

## Core Rules

### Feature boundaries

- Give each feature a folder or slice that is easy to find and reason about.
- Keep the code, tests, and local assets that change together close together.
- When a feature has its own folder, add a short `README.md` there that explains what the feature owns, where the main entrypoints live, and how to validate it.
- Avoid letting one feature depend directly on another feature's internals.

### Shared code

- Create shared utilities only when reuse is real and stable.
- Do not create a global `utils` or `helpers` bucket unless it has clear internal categories.
- Prefer duplication of tiny stable code over premature shared abstractions that create coupling.

### Product code vs maintenance scripts

- If code is a user-facing feature of the product or package, keep it under the main source tree.
- If code is repo maintenance, migration, scaffolding, or one-off automation, keep it in `scripts/` or the equivalent maintenance area.
- If an installed CLI needs a runtime entry file, keep the implementation under the owning feature in `src/` and let `scripts/` hold only the smallest executable shim that calls into that feature.
- Do not leave product behavior buried in maintenance scripts just because it started as a prototype.

### Mixed Python and JavaScript or TypeScript setup

- When a repo ships both Python and Node surfaces, let each language keep its normal packaging and config entrypoints while sharing the same feature names and folder ownership.
- Prefer colocated `main.py` and `main_test.py` for Python plus the matching Node files for the same feature when both runtimes expose that surface.
- For no-build npm packages or Git-installed CLIs that run from `node_modules`, prefer `.mjs` plus JSDoc, such as `main.mjs` and `main.test.mjs`, so the package does not rely on TypeScript runtime stripping.
- When TypeScript is the selected Node surface and it is not shipped as directly executed package runtime, use `main.ts` and `main.test.ts` for the same feature.
- Keep Python packaging in `pyproject.toml`, keep Node packaging in `package.json`, and split language-specific config such as `tsconfig` files by runtime surface instead of forcing one config file to describe everything.
- Keep shared cross-runtime helpers narrow and explicit so a feature does not quietly depend on another language surface's internals.

### Naming and discoverability

- Choose folder names that match the domain or workflow they contain.
- Keep file names descriptive enough that a reader can predict the contents before opening them.
- Avoid over-generic folder names when a narrower domain name exists.
- When sibling directories or files form a sequence rather than independent peers, prefix each name with a fixed-width number so the natural sort matches the intended order — `10-new/`, `20-open/`, `90-closed/`.

**Where the numeric-prefix convention comes from.** It is the Linux `.d`-directory pattern, inherited
from SysV init (`/etc/rc3.d/S20apache2`) and formalised by systemd's drop-in configuration
directories. `man 5 sysctl.d` states the rule: configuration files are "sorted by their filename in
lexicographic order," and it "is recommended to prefix all filenames with a two-digit number and a
dash." The number is not metadata — nothing parses it — it exists only so that string sorting yields
the intended order. That mechanism dictates the practices: pad to a fixed width, because `9-` sorts
*after* `10-` under string comparison; leave gaps (step 10) so a later insertion never forces a
renumber; and reserve the top of the range for local overrides, the near-universal `99-` idiom. Apply
it only where order changes meaning — `/etc/modprobe.d` and `/etc/cron.d` are deliberately unnumbered
because their entries are independent. One caveat when borrowing it: `.d` numbers encode *priority
among simultaneous entries*, while a lifecycle encodes a *sequence*. The sorting trick is shared, the
semantics are not, so do not import priority idioms such as a `50-` neutral centre into a sequence.

### Growth path

- Start small, but keep a path open for splitting files as complexity grows.
- Extract pure helpers before extracting stateful orchestration.
- If a file is hard to scan, split by responsibility inside the same feature before introducing repo-wide infrastructure.

## Example Layouts

### Python package with feature-first layout

```text
scripts/ (only for devs, not installed with the package)
  data-migration.py
  generate-docs.py
src/package_name/
  billing/
    main.py
    main_test.py
    service.py
    service_test.py
```

### Mixed Python and no-build Node package

```text
scripts/
  billing-cli.mjs
src/package_name/
  billing/
    main.py
    main_test.py
    main.mjs
    main.test.mjs
  utils/
    common.mjs
```

### Standalone browser tool with local assets

```text
scripts/ (only for devs, not installed with the package)
  data-migration.mjs/mts
  generate-docs.mjs/mts
src/features/billing/
  index.html
  app.js
  styles.css
```

## Validation

- A new engineer can find the owning feature quickly.
- Product code is not hidden inside maintenance paths.
- Shared utilities exist because of real reuse, not prediction.
- Feature boundaries are explicit and avoid backdoor dependencies.
- Tests live close enough to their source that maintenance stays cheap.
- Ordered sibling names use a fixed-width numeric prefix, so a plain lexicographic listing shows them in their real order.

## References

## Checklist

## Review Checklist

- Product code lives in feature-owned paths instead of maintenance folders.
- Shared utilities exist because of real reuse, not speculative abstraction.
- Feature boundaries are easy to identify and avoid circular dependencies.
- In multi-package repos, each package owns its own `src/` tree instead of reaching through peer-package internals.
- Scripts, migrations, and repo automation stay separate from shipped product code.
- Tests sit close enough to the owning feature that maintenance cost stays low.

---

# ref-spst-dev-code-conventions

_When this applies: SwiftPost Site Template TypeScript and React code quality standards. Use when: creating components, writing hooks, reviewing TypeScript code, or applying template-specific coding patterns._

# SwiftPost Code Conventions

## Purpose

Enforce consistent TypeScript and React patterns across the monorepo, keeping code simple, strict, readable, and maintainable. Favor obvious code over clever code and prefer structures that stay easy to change.

## When to use this skill

- Creating or modifying React components or hooks.
- Writing TypeScript utility functions or types.
- Reviewing code for type safety and pattern compliance.

## TypeScript Rules

### Strict Typing

- **No `any`:** Strictly banned. Use proper typing, generics, or `unknown` combined with type guards.
- **Strict mode:** The project uses `"strict": true` in tsconfig. Never weaken it.
- **Erasable types:** Prefer erasable syntax. No `enum` (use `as const` objects or union types), no `namespace`, no parameter properties. Use `type` imports (`import type { ... }`) when importing types only.

### Type Inference

- **Let return types be inferred** in most cases. TypeScript's inference is sound — trust it.
- **Explicitly annotate return types** only when defining an API contract (e.g., a service function, a hook's public return shape, or a shared utility that others depend on).
- **Always type function parameters.** Return types can be inferred; input types cannot.

### Type Declarations

- Use `type` over `interface` when the type is not extended. Prefer `type Props = { ... }` for component props that won't be inherited.
- Use `interface` when you need declaration merging or expect extension (e.g., slot prop interfaces).
- Keep types close to usage. Only move to a shared `types.ts` if genuinely reused across multiple files.

## React Component Rules

### Structure

```tsx
import Box from '@swiftpost/elysium/ui/base/Box';
import Text from '@swiftpost/elysium/ui/base/Text';

interface Props {
  title: string;
  children: React.ReactNode;
}

const MyComponent: React.FC<Props> = ({ title, children }) => {
  return (
    <Box>
      <Text>{title}</Text>
      <Box>{children}</Box>
    </Box>
  );
};

export type MyComponentProps = Props;
export default MyComponent;
```

### Rules

- **Const functions:** Always use `const` arrow functions. No `function` declarations for components or utilities.
- **`React.FC<Props>`:** Always use this pattern for component definitions.
- **Props interface:** Always define `interface Props` (omit only if the component takes no props).
- **Export pattern:** Export `type {ComponentName}Props = Props` at the bottom. Default-export the component.
- **Early returns:** Prefer early returns to reduce nesting and keep conditional logic easy to follow.

```tsx
// Good — early return
const UserCard: React.FC<Props> = ({ user }) => {
  if (!user) {
    return null;
  }

  return <Text>{user.name}</Text>;
};

// Bad — nested conditional
const UserCard: React.FC<Props> = ({ user }) => {
  return (
    <Box>
      {user ? <Text>{user.name}</Text> : null}
    </Box>
  );
};
```

## Custom Hook Rules

Hooks must be resilient, typed, and encapsulate their own side effects.

- **Return objects:** Return `{ value, setValue }` instead of tuples for extensibility and readability.
- **Error handling:** Silent failure via `try/catch` is required for browser APIs (e.g., `sessionStorage`, `localStorage`) to prevent hydration crashes.
- **Cleanup:** Always clean up side effects in `useEffect` returns (e.g., `removeEventListener`).
- **Const functions:** Hooks are also const arrow functions.

```tsx
import { useState, useCallback, useEffect } from 'react';

export const useCustomFeature = <TValue,>(initialValue: TValue) => {
  const [value, setInternalValue] = useState<TValue>(initialValue);

  const setValue = useCallback((newValue: TValue) => {
    try {
      setInternalValue(newValue);
    } catch (error) {
      console.error('Feature execution failed:', error);
    }
  }, []);

  return { value, setValue };
};
```

## General Patterns

- **No unnecessary libraries.** Before proposing a new dependency, always ask the user and do a thorough check for alternatives. Write a few lines of TypeScript instead of pulling in a micro-library.
- **Use approved libraries when a dependency is justified.** Prefer `@tanstack/react-query` for async server state, `zod` for schema validation, and `next-intl` for internationalization. Prefer native `Date` and `Intl.DateTimeFormat` for normal date work; use `date-fns` only when the problem genuinely needs more complex date math. Any other dependency still requires explicit user approval after checking for simpler alternatives.
- **Prefer modern APIs.** Use intention-revealing built-ins like `find`, `some`, `every`, `includes`, `Object.hasOwn`, and `at` when they express the behavior directly. Avoid older sentinel-style patterns like `indexOf(...) !== -1` or manual loops when a modern built-in is clearer and equally supported.
- **Curly braces required.** All `if`/`else`/`for`/`while` blocks must use curly braces (enforced by ESLint `curly: ['error', 'all']`).
- **No useless renames.** Destructuring renames must add meaning (enforced by ESLint `no-useless-rename`).
- **Unused variables.** Prefix with `_` if intentionally unused (e.g., `_event`, `_index`).

## Standalone Scripts

Scripts in `scripts/` use `.mts` (TypeScript ES Modules) and run via Node's native type stripping (`node --experimental-strip-types`). This requires **Node >= 22.6**.

### Rules

- All project TypeScript rules apply: no `any`, strict types, `const` arrow functions, `import type` for type-only imports.
- Use `interface` for local type definitions (not `type` — keeps consistency with the rest of the codebase for structured shapes).
- Return type annotations are optional for internal helpers; explicit for exported/public API functions.
- Use `node:` protocol for Node built-in imports (`import fs from 'node:fs'`).
- Do not use `/// <reference types="node" />` in script files. Standalone scripts must live under a tsconfig that provides Node types via `compilerOptions.types`.

### Example

```ts
import type { PathLike } from 'node:fs';
import fs from 'node:fs';

interface Config {
  name: string;
  values?: string[];
}

const readConfig = <T>(filePath: PathLike, fallback: T): T => {
  if (!fs.existsSync(filePath)) {
    return fallback;
  }
  return JSON.parse(fs.readFileSync(filePath, 'utf-8')) as T;
};
```

---

# ref-spst-dev-config-package

_When this applies: SwiftPost shared config package overview. Use when: editing packages/config, adjusting shared ESLint or TypeScript config, or deciding whether tooling rules belong in the config package._

# SwiftPost Config

## Purpose

Clarify the role of `packages/config` as the shared tooling package for linting and TypeScript configuration. Use this skill when a change affects reusable repo tooling rather than application behavior.

## When to use this skill

- Editing files in `packages/config`.
- Changing shared ESLint or TypeScript defaults.
- Deciding whether a tooling rule belongs in package config or in an app/package implementation.

## Package Responsibility

`packages/config` owns reusable lint and TypeScript configuration for the monorepo.

- Keep shared ESLint config in `eslint.config.mjs` and `eslintBaseConfig.mjs`.
- Keep shared TS defaults in `tsconfig.json` and `tsconfigBase.json`.
- Do not put app logic, UI code, or Next.js page behavior in this package.

## Change Guidelines

- Favor changes that benefit multiple packages rather than one-off app workarounds.
- If a rule is only needed by one package, prefer solving it in that package unless it clearly belongs in the shared baseline.
- Keep config changes explicit and easy to trace because they affect the entire repo.
- For flat ESLint config files, prefer ESLint's `defineConfig` helper from `eslint/config` and consume `typescript-eslint` via named config exports instead of relying on `tseslint.config()`.

For general code-style guidance, see the `ref-spst-dev-code-conventions` skill. For repo-wide structure, see the `ref-spst-dev-site-architecture` skill.

---

# ref-spst-dev-main-package

_When this applies: SwiftPost main app package overview. Use when: working in packages/main, placing app code, understanding package boundaries, or deciding whether logic belongs in the app package versus another package._

# SwiftPost Main

## Purpose

Clarify the role of `packages/main` as the deployed Next.js app package. Use this skill for package-specific boundaries, entry points, and responsibilities that are specific to SwiftPost's application shell.

## When to use this skill

- Adding or moving code inside `packages/main`.
- Deciding whether logic belongs in the app package or another package.
- Working on route entry points, shared app components, templates, or app-local theme wrappers.

## Package Responsibility

`packages/main` is the deployable Next.js application. It should compose UI and features, not duplicate shared infra that belongs elsewhere.

- Put shared lint and TS config concerns in `@swiftpost/config`.
- Put shared UI primitives and wrappers in `@swiftpost/elysium`.
- Keep app-specific pages, templates, components, and app wiring in `packages/main`.

## Main Entry Points

- `src/app/` for App Router pages and layouts.
- `src/components/` for shared app-level presentational components.
- `src/templates/` for reusable page templates.
- `src/styles/` for app-local theme wrappers like `staticTheme`.
- `next.config.ts` for static export and app deployment settings.

## Package-Specific Notes

- The app depends on `@swiftpost/elysium` for UI and `@swiftpost/config` for tooling.
- `next.config.ts` sets `output: 'export'` and a production `basePath` for deployment.
- Prefer thin route entry points and keep reusable UI logic out of the route file when possible.

For framework-wide rules that can apply to other Next.js projects, see the shared `ref-sp-js-next` skill. For this repo's structure, see the `ref-spst-dev-site-architecture` skill.

---

# ref-spst-dev-site-architecture

_When this applies: SwiftPost Site Template architecture, monorepo layout, package boundaries, import rules, and feature placement. Use when: designing features, structuring components, understanding where code goes, or configuring template-specific tooling._

# Architecture

## Purpose

Define the high-level architectural rules for the repo: how code is organized, how components are composed, how client/server boundaries work, how features stay isolated, and where files go. Favor structures that keep the codebase simple to navigate and maintain over abstractions that add indirection without clear payoff.

For the SwiftPost Elysium UI library reference (components, props, imports, styling helpers), see the **ref-spst-js-elysium** skill.
For SwiftPost-specific styling guidance, see the **ref-spst-js-styling** skill.
For TypeScript/React coding patterns, see the **ref-spst-dev-code-conventions** skill.
For Next.js-specific constraints and page patterns, see the **ref-spst-js-next** skill.

## When to use this skill

- Designing a new feature or component hierarchy.
- Deciding where business logic vs. presentation belongs.
- Understanding where to place new code.
- Setting up a new feature module.
- Navigating the monorepo layout.
- Reviewing modularity and separation of concerns.

## Monorepo Overview

**Turborepo** with **Yarn workspaces**. Three packages:

| Package | Path | Purpose |
|---------|------|---------|
| `@swiftpost/config` | `packages/config/` | Shared ESLint and TypeScript configs |
| `@swiftpost/elysium` | `packages/elysium/` | Internal UI library — thin MUI 7 wrappers + enhanced components |
| `@swiftpost/main` | `packages/main/` | Next.js 15 app (static export, App Router, Turbopack) |

## Local Agent Workspaces

The `.agents/playground/`, `.agents/tasks/`, and `.agents/retro/` directories are local-only agent workspaces. They are ignored by Git and excluded from AI context through `.ai-policy.json`. Each keeps a committed placeholder `.gitignore` so the directory survives a clone.

| Path | Purpose | Rule |
|------|---------|------|
| `.agents/playground/` | Temporary helper scripts, scratch files, generated local artifacts, and other short-lived agent work. | Do not put committed source, durable documentation, or secrets here. Promote anything reusable into the proper package, script, doc, or skill. |
| `.agents/tasks/` | Local task tracking, backlog notes, task briefs, validation notes, and temporary planning artifacts. | Keep it local and current. Promote durable decisions into committed docs or skills instead of relying on ignored notes. See `ref-sp-agents-local-tasks` for the lifecycle. |
| `.agents/retro/` | Retrospectives captured at the end of substantial work. | Read past retros before similar work. See `ref-sp-agents-retro` for the format. |

This repository follows the portable `.agents/` layout directly, so no path translation is needed when a shared skill refers to it.

## `packages/main/src/` Directory Map

```
src/
├── app/                          # Next.js App Router pages
│   ├── page.tsx                  # Home page
│   ├── layout.tsx                # Root layout
│   ├── globals.css               # Global styles
│   └── <route>/                  # Additional routes
│       ├── page.tsx              # Thin server component (metadata)
│       └── ClientWrapper.tsx     # 'use client' boundary
├── components/                   # Shared layout/presentational components
│   ├── Footer.tsx
│   ├── Header.tsx
│   ├── Logo.tsx
│   ├── Menu.tsx
│   └── TopBar.tsx
├── features/                     # Domain feature modules (feature-first)
│   └── <feature-name>/
│       ├── index.ts              # Barrel file — re-exports public API
│       ├── types.ts              # Zod schemas + inferred TS types
│       ├── constants.ts          # Default data, config values
│       ├── hooks/                # Feature-specific hooks
│       ├── components/           # Feature-specific UI components
│       ├── services/             # Data access / business logic
│       └── utils/                # Feature-specific helpers
├── styles/                       # Theme configuration
│   ├── theme.ts
│   └── staticTheme.ts
├── templates/                    # Page layout templates
│   ├── SimplePageTemplate.tsx
│   └── BlogPostTemplate.tsx
├── customConfig.ts
└── types.ts                      # Shared types (only if truly cross-feature)
```

## Modularity & Feature Isolation

Fat components are strictly banned. UI components must be presentation-focused.

* **The `features` Directory:** Domain-specific logic (e.g., `storage`, `auth`, `data-processing`) must be encapsulated within `packages/main/src/features/{feature-name}/`. Expose hooks, types, and constants from here. Do not leak business logic into `src/components`.
* **Component Splitting:** Extract complex UI states (e.g., Authenticated vs. Unauthenticated views) into private, sibling components within the same file to keep the main render method clean and readable.
* **No cross-feature imports:** Features must not import from other features. Shared logic goes in `src/components/` (presentation) or a shared utility.

## Feature-First Architecture

Domain logic lives in `packages/main/src/features/<feature-name>/`. Features are self-contained modules that encapsulate their own types, hooks, components, services, and utilities.

### Feature Structure

```
features/<feature-name>/
├── index.ts              # Barrel — re-exports the feature's public API
├── types.ts              # Zod schemas + inferred TS types
├── constants.ts          # Config values, defaults
├── hooks/
│   ├── index.ts          # Barrel for hooks
│   ├── useFeatureData.ts
│   └── useFeatureState.ts
├── components/
│   ├── index.ts          # Barrel for components
│   ├── FeatureDashboard.tsx
│   └── FeatureCard.tsx
├── services/
│   └── featureService.ts # Plain object singletons with async methods
└── utils/
    └── formatters.ts     # Feature-specific helpers
```

### Feature Rules

- **Barrel files:** Every feature root and major subfolder has an `index.ts` that re-exports the public API.
- **Self-contained:** Features must not import from other features. Shared logic goes in `src/components/` or a shared utility.
- **No business logic in components:** `src/components/` is presentation-only. Domain logic stays in `src/features/`.
- **Services are plain objects:** Use object singletons with async methods, not classes.
- **Zod for domain models:** Define schemas in `types.ts`, derive TS types with `z.infer<>`.

## Composition Guidelines

* **Presentation vs. Logic:** Components in `src/components/` are presentation-only shells. They receive data via props and render UI. All data fetching, state management, and business logic belongs in `src/features/`.
* **Hook Encapsulation:** Domain logic exposed to components should be wrapped in custom hooks that live in `src/features/{feature}/hooks/`. Components consume these hooks — they don't implement the logic inline.
* **Barrel Exports:** Every feature root and major subfolder has an `index.ts` that re-exports the public API. Consumers import from the barrel, not from internal files.

## Reusable Component Architecture

Reusable components that are meant to be flexible and overridable must use the Slots & SlotProps pattern. See the **ref-spst-js-styling** skill for the local styling pattern and **ref-spst-js-elysium** for the concrete package implementation used here.

Key rules:
* Define `SlotProps` for internal elements, `Props` with `slots?`, `slotProps`, and `sx?`.
* Never overwrite `sx` — always merge with `spreadSx`.
* Use `const componentBaseName` (kebab-case) for CSS class targeting.
* Wrap reusable UI components in `memo`.

## Import Rules

Do not guess import paths. Adhere strictly to these conventions:

| What | Import From |
|------|-------------|
| MUI base components | `@swiftpost/elysium/ui/base/{ComponentName}` |
| Enhanced components (Link, Image, etc.) | `@swiftpost/elysium/ui/{ComponentName}` |
| MUI Icons | `@mui/icons-material` |
| Shared layout components | `@/components/{ComponentName}` |
| Feature modules | `@/features/{feature-name}` or `@/features/{feature-name}/{subpath}` |
| Theme values (SSR-safe) | `@/styles/staticTheme` |
| Links/Navigation | `@swiftpost/elysium/ui/Link` (never `next/link` directly) |

## File Placement Guide

| Scenario | Where | Why |
|----------|-------|-----|
| New domain feature | `src/features/<name>/` | Feature-first isolation |
| Shared layout component | `src/components/` | Presentation-only, cross-feature |
| New route/page | `src/app/<route>/page.tsx` + `ClientWrapper.tsx` | App Router convention |
| Page template | `src/templates/` | Reusable page layouts |
| Theme config | `src/styles/` | Centralized theming |
| Cross-feature types | `src/types.ts` | Only if genuinely shared |
| Feature-specific types | `src/features/<name>/types.ts` | Keep close to usage |
