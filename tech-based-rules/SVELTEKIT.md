# SVELTEKIT.md

Conventions for AI agents working in SvelteKit codebases. This is a reusable baseline for every SvelteKit project: full-stack applications with a database and authentication, content sites, and fully static sites. Project-specific choices go in "Project Settings"; everything else holds for every project.

**Ground rules:**

1. **The tooling is the source of truth.** `svelte-check`, ESLint, Prettier and `tsconfig.json` define the enforceable rules. Never introduce code that fails them.
2. **Always target the latest stable Svelte and SvelteKit.** Verify APIs against the official documentation at svelte.dev instead of relying on memory. Svelte 5 replaced most of the component API, so older patterns from training data are wrong.
3. **Fill in "Project Settings" per project.**

## Versioning

- Build on the latest stable Svelte (5 or later) and SvelteKit, with compatible Node and TypeScript versions.
- Scaffold new projects with the official `sv` CLI (`pnpm dlx sv create`) and add integrations with `sv add` (for example `drizzle`, `paraglide`, `vitest`, `playwright`, `eslint`, `prettier`) instead of wiring them by hand. Verify the available add-on names in the `sv` documentation.
- Experimental features (options under `kit.experimental` or `compilerOptions.experimental`) are not used unless Project Settings enable them.
- The package manager is pnpm.

## Project Settings (fill in per project)

> Replace these placeholders for the current project, then delete this note.

- **Adapter:** `<adapter-node (server) | adapter-static (fully static)>`.
- **Deployment:** `<Docker on Coolify | static hosting | other>`.
- **Database:** `<none | SQLite | PostgreSQL>`, driver `<e.g. better-sqlite3 | libsql | postgres>`.
- **Authentication:** `<none | Better Auth>`, roles `<list>`.
- **Locales:** `<e.g. en, tr>`, default `<e.g. en>`, URL strategy `<prefix all locales | prefix all except default>`.
- **Prerendering:** `<pages to prerender | all>`.

## Commands

- `pnpm dev`: development server.
- `pnpm build`, then `pnpm preview`: production build and local preview.
- `pnpm check`: `svelte-check` type and accessibility diagnostics.
- `pnpm lint` and `pnpm format`.
- `pnpm test:unit` (Vitest) and `pnpm test:e2e` (Playwright).

Adjust to the project's actual `package.json` scripts. Per [GENERAL-RULES.md](../GENERAL-RULES.md), outside an autonomous run agents do not run these commands or start long-running processes; give the user the exact command instead.

## Formatting

- Prettier with `prettier-plugin-svelte`, configured with `"useTabs": true` and `"tabWidth": 4` (GENERAL-RULES.md).
- The HTML rule in GENERAL-RULES.md (no blank lines between tags) applies to Svelte markup.

## Project Structure

```text
src/
├── app.d.ts                 # App.Locals, App.PageData, App.Error types
├── app.html
├── hooks.server.ts          # session, locale, security headers
├── lib/
│   ├── components/          # reusable components, grouped by domain
│   ├── server/              # server-only code
│   │   ├── auth.ts
│   │   ├── db/
│   │   │   ├── index.ts     # database client
│   │   │   └── schema.ts    # Drizzle schema
│   │   └── services/
│   ├── state/               # shared reactive state (*.svelte.ts)
│   ├── types/               # shared TypeScript types
│   └── utils/               # pure helpers
├── routes/
└── styles/                  # global SCSS: tokens, reset, typography, mixins
messages/                    # Paraglide message files, one per locale
drizzle/                     # generated migrations, committed
tests/e2e/                   # Playwright tests
```

- Anything that touches the database, secrets or private environment variables lives under `src/lib/server/`. SvelteKit refuses to bundle it into client code; never work around that.
- Shared types go in `src/lib/types/`, which replaces the `modules/interfaces/` default in GENERAL-RULES.md. Types used by one file live next to it. Database row types are inferred from the Drizzle schema, never duplicated by hand.
- Unit tests sit next to the code they test as `*.test.ts`. End-to-end tests live in `tests/e2e/`.

## Naming Conventions

- Components: PascalCase file names (`PostCard.svelte`).
- Other TypeScript files and folders: kebab-case (`date-format.ts`). Rune-based state modules end in `.svelte.ts`.
- Route folders: kebab-case, matching the URL.
- Variables, functions and properties: camelCase. Types, interfaces and enums: PascalCase. SCSS variables: camelCase (GENERAL-RULES.md).

## Components and Reactivity

- Use Svelte 5 runes: `$state`, `$derived`, `$effect`, `$props`, `$bindable`. Never use legacy syntax: no `export let`, no `$:` statements, no `on:event` directives, no `<slot>`, no `createEventDispatcher`.
- Values computed from other state use `$derived`. `$effect` is only for side effects (DOM APIs, subscriptions, third-party libraries); never use it to copy one piece of state into another.
- Type props with an interface: `let { title, items = [] }: Props = $props();`.
- Event handlers are attributes (`onclick={handleClick}`). Components communicate upward through callback props.
- Pass markup into components with snippets and `{@render}`.
- Shared state lives in `.svelte.ts` modules under `src/lib/state/`. Use stores only where a library requires them.
- Keep components focused. When a component mixes data access, business logic and presentation, split it.

### Server-side state safety

**Never keep per-user or per-request data in module-level variables** in code that runs on the server. The server process is shared by every visitor, so module state leaks between users. Pass request-scoped data through `event.locals`, `load` return values or Svelte context (`setContext` and `getContext`).

## Templates

- Conditional blocks use `{#if}` and `{:else}`. Inline value selection inside a template expression may use the ternary operator (`class={active ? 'tab active' : 'tab'}`). This is the only exception to the ternary rule in GENERAL-RULES.md, and it does not extend to `<script>` blocks.
- Lists use keyed each blocks: `{#each items as item (item.id)}`.
- `{@html}` is only allowed with content sanitized on the server, never with raw user input.

## Routing and Data Loading

- Data needed for the first render comes from `load` functions, never from fetches in `onMount`.
- `+page.server.ts` and `+layout.server.ts` for anything that needs the database, secrets or the session. `+page.ts` only for public data that can load on both server and client.
- Inside `load`, use the provided `fetch`, not the global one.
- Signal failures with `error()` and `redirect()` from `@sveltejs/kit`.
- Return only what the page renders. Never send fields the client must not see (password hashes, tokens, internal flags).
- API endpoints (`+server.ts`) are for external consumers: other apps, webhooks, API keys. The application's own pages use `load` and form actions.

## Forms and Validation

- Mutations use form actions with progressive enhancement (`use:enhance`). Pages work without JavaScript unless Project Settings say otherwise.
- Validate every input on the server with Zod, even when the client validates too. Return failures with `fail(400, { ... })`, including the submitted values and field errors.
- Derive validation schemas from the Drizzle schema where it fits (`drizzle-zod`).

## Database (Drizzle ORM)

- Drizzle ORM is the only database access layer. Raw SQL is only allowed through Drizzle's `sql` template tag, never built by string concatenation.
- The schema in `src/lib/server/db/schema.ts` (split per domain when it grows) is the single source of truth for tables and types.
- Schema changes go through `drizzle-kit generate`. Generated migrations in `drizzle/` are committed and never edited after they have been applied. `drizzle-kit push` is not used.
- Applying migrations follows the database rules in GENERAL-RULES.md: outside an autonomous run, give the user the exact command.
- Writes that must succeed or fail together run in a transaction.

## Authentication (when enabled)

- Better Auth with its Drizzle adapter. Its tables are part of the Drizzle schema and migrations.
- Resolve the session once in `hooks.server.ts` and expose it on `event.locals`, typed in `app.d.ts`.
- Authorization is enforced on the server: hooks, server `load` functions, actions and endpoints. Hiding a button is never access control.

## Environment and Configuration

- Private values come from `$env/static/private` or `$env/dynamic/private`, public values from the matching `public` modules and carry the `PUBLIC_` prefix.
- With adapter-node and Docker, values that change per deployment use `$env/dynamic/*`, which is read at runtime. `$env/static/*` is baked in at build time and only fits values that never change for a build.
- Every variable is listed in `.env.example` with a comment. `.env` is never committed.

## Internationalization (standard)

- Paraglide JS is the i18n layer. Messages live in `messages/<locale>.json` with stable, namespaced keys.
- **Never hardcode user-facing strings** in markup or TypeScript. This includes page titles, meta descriptions, alt texts and validation messages.
- Every locale is complete: a key added to one message file is added to all of them.
- Localized pages declare `hreflang` alternates and the correct `lang` attribute.

## Styling and Theming (light and dark are standard)

- SCSS everywhere: component styles in `<style lang="scss">`, global styles in `src/styles/`. Use `@use` and `@forward`, never the deprecated `@import`.
- Theme values are CSS custom properties defined on `:root` and overridden under `[data-theme='dark']`. SCSS variables are only for build-time constants that do not change with the theme (breakpoints, spacing scale). Never hardcode colors in components.
- Initial theme follows `prefers-color-scheme`. An explicit user toggle persists in a cookie so the server renders the right theme, and `data-theme` is set before first paint to prevent a flash of the wrong theme.
- Component styles stay scoped. `:global()` is only used in `src/styles/` or for content the component does not own (for example rendered rich text), always nested under a component class.
- The SCSS rules in GENERAL-RULES.md apply (camelCase variables, `>.child` without a space).

## Images and Performance

- Local images use `@sveltejs/enhanced-img` (`<enhanced:img>`) for responsive formats and sizes.
- Images from a CMS or from uploads get explicit `width` and `height`, `loading="lazy"` below the fold, and a responsive `srcset` when the source can resize them.
- Prerender every page that does not depend on the request.
- Every page sets its own `<title>` and meta description in `<svelte:head>`.

## Accessibility

- `svelte-check` accessibility warnings are fixed, not suppressed. A `svelte-ignore` is only acceptable with a comment explaining why the warning is a false positive.
- Interactive elements are real `<button>` and `<a>` elements, never clickable `<div>` elements.

## Testing (standard)

- New logic comes with Vitest unit tests. Every user-facing flow has a Playwright end-to-end test.
- Tests never depend on a real external service. Use a separate test database and mocks.
- Running tests follows GENERAL-RULES.md: outside an autonomous run, give the user the command.

## Anti-patterns: do NOT

- Use legacy Svelte syntax (`export let`, `$:`, `on:click`, `<slot>`, `createEventDispatcher`).
- Use `$effect` to derive state.
- Fetch initial page data in `onMount`.
- Import server-only modules or private environment variables into client code.
- Store per-user data in module-level variables on the server.
- Build SQL by string concatenation or edit applied migrations.
- Hardcode user-facing strings or colors.
- Use `{@html}` with unsanitized content.
- Use experimental features without a Project Settings entry.

## Definition of Done

1. `pnpm check` and `pnpm lint` report no errors or warnings.
2. `pnpm build` succeeds.
3. Unit and end-to-end tests for the change exist and pass.
4. Every new user-facing string exists in every locale.
5. The change works in the light and the dark theme.
6. No new `any`, no private data sent to the client, no module-level request state.
