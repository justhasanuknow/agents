# ANGULAR.md

Conventions for AI agents working in this Angular codebase. This is a reusable, future-proof baseline for modern Angular projects.

**Ground rules:**

1. **The tooling is the source of truth.** ESLint, Prettier, and `tsconfig.json` define the enforceable rules. When in doubt, run the linter and fix what it reports. Never introduce a pattern that fails `ng lint`.
2. **Always target the latest stable Angular.** See Versioning below. Do not assume a fixed version number.
3. **Fill in "Project Settings" per project.** Everything else is framework-level guidance that holds regardless of project.

## Versioning

- **Always build on the latest stable Angular release**, with a Node and TypeScript version compatible with it. Do not pin to an older major out of habit.
- At project start, check the current stable version (e.g. `npm view @angular/core version`, or the official Angular releases page) and scaffold with the matching CLI (`npm create @angular@latest` / `ng new`). When upgrading, use `ng update`, not manual edits.
- Treat the **current version's recommended/default features as the baseline**: standalone components, the built-in control flow (`@if` / `@for` / `@switch`), and the `inject()` function. Exception: **signals are not used in this baseline** (see Component Conventions). Reactive state is handled with observables. Verify version specifics against the official docs rather than assuming.
- Keep dependencies on a supported (non-EOL) major; Angular majors get ~18 months of support.

## Project Settings (fill in per project)

> Replace these placeholders for the current project, then delete this note.

- **Package manager:** `<npm | pnpm | yarn>`.
- **Styling:** `<SCSS | CSS>` + UI library `<none | Angular Material | ... >`. (Light + dark theming is standard, see Styling & Theming.)
- **State approach:** `<RxJS services | NgRx>`. Pick one and apply consistently. Signals are not used (see Component Conventions).
- **Change detection:** `<OnPush everywhere | framework default>`.
- **Routing:** `<path location | hash location>`, lazy loading `<yes | no>`.
- **i18n:** library `<@ngx-translate | @angular/localize>` + supported languages `<e.g. en, tr>`, default/fallback `<e.g. en>`. (Multi-language is standard, see Internationalization.)
- **Lint thresholds** (defined in ESLint, not here): max function length, max file length, max line length.

## Commands

- `npm start` / `ng serve`: dev server.
- `ng build`: production build (verify before declaring a task done).
- `ng test`: unit tests.
- `ng lint`: lint TypeScript and templates.

Adjust to the project's actual `package.json` scripts.

Per [GENERAL-RULES.md](../GENERAL-RULES.md), agents do not run `npm` / `node` commands and do not start long-running processes (`ng serve`, watch modes). Give the user the exact command to run and ask them to run it; the same applies to the `npm view` / `ng new` commands in Versioning above.

## Project Structure

Keep a predictable, role-based layout under `src/app/`. A common shape:

- `pages/` (or `features/`): routed feature components, nested to mirror the route hierarchy. A "page" is anything reachable via the router.
- `shared/`: reusable, app-wide UI components, directives, and pipes used across features.
- `core/` (or a `public/`-style folder): non-component shared code, with **one folder per kind**:
  - `classes/`: framework-agnostic helpers.
  - `guards/`, `interceptors/`, `resolvers/`, `directives/`, `pipes/`: one Angular artifact per file.
  - `interfaces/` (or `models/`): TypeScript types, sub-foldered by domain when they grow.
  - `enums/`, `types/`: shared enums and type aliases.
  - `constants/`: default/constant data sets.
- `services/`: services grouped by domain (e.g. `api/`, `state/`, plus feature-specific groups).
- `styles/`: global styles and theme tokens (see Styling & Theming).
- `src/environments/`: environment configuration (see below).
- A dedicated location for translation files (see Internationalization).

Rules of placement:

- Routed component → `pages/`. Generic reusable widget → `shared/`. Cross-cutting logic with no template → `core/`.
- **Prefer extending an existing shared component over duplicating one.** If more functionality is needed, develop the existing component further rather than creating a near-copy.

## Environment & Configuration

- Keep environment files minimal and parallel: `environment.ts` (**production / default**) and `environment.development.ts`. The Angular builder swaps them via `fileReplacements` in `angular.json` under the `development` configuration.
- **Never import `environment.development.ts` directly.** Always import the base file; the build substitutes it:

  ```ts
  import { environment } from '../../environments/environment';
  ```

- All environment files must export an object with **identical keys**. When you add a config value, add it to **every** environment file with the same key, otherwise the production build fails to type-check.
- **Never hardcode API/socket/base URLs** in components or services; read them from `environment`.
- Register application-wide providers (router, HTTP client + interceptors, i18n, etc.) in `app.config.ts`, not in individual components.
- Never commit secrets or tokens. Environment files hold non-sensitive config (URLs, flags) only; credentials belong on the backend.

## Naming Conventions

- **Files:** kebab-case with an Angular type suffix: `*.component.ts/.html/.scss`, `*.service.ts`, `*.guard.ts`, `*.interceptor.ts`, `*.directive.ts`, `*.pipe.ts`, `*.resolver.ts`.
- **Model/type files:** group related types per domain (e.g. `auth-models.ts`), rather than one file per interface.
- **Classes / interfaces / enums / type aliases:** PascalCase. **Variables / properties / methods:** camelCase.
- **Selectors:** component selectors are element-type, kebab-case, with a project prefix (e.g. `app-user-card`); directive selectors are attribute-type, camelCase, same prefix. Enforce the prefix via ESLint.
- **Functional guards/interceptors/resolvers:** export as camelCase `const` arrow functions (e.g. `authGuard`, `apiInterceptor`). Keep this consistent across the project.
- **Unused-but-required parameters:** prefix with `_` (e.g. `(_error) => ...`).

## Component Conventions

- Always `standalone: true`. Declare every dependency in the component's `imports` array; do not rely on shared NgModules.
- Use external `templateUrl` and `styleUrl` files (not inline) unless the template is trivial.
- **Inject dependencies with the `inject()` function**, not constructor parameters:

  ```ts
  private userService = inject(UserService);
  ```

- **Do not use signals when creating components.** Use observables (RxJS) for reactive state, and manage change detection explicitly: inject `ChangeDetectorRef` and call `markForCheck()` / `detectChanges()` when needed.
- Use the `@Input()` / `@Output()` decorators for inputs and outputs. Do not use the signal-based `input()` / `output()` functions.
- Recommended: `changeDetection: ChangeDetectionStrategy.OnPush` for predictable, performant rendering, paired with the explicit `ChangeDetectorRef` handling above.
- **Do not create `*.spec.ts` files.**
- Keep components thin: data fetching and business logic live in services, not components. Put route-snapshot-dependent setup in the constructor and other init in `ngOnInit`.

## Services & State

- Services are `@Injectable({ providedIn: 'root' })` singletons unless they are intentionally scoped.
- Organize services by domain (API access, state, feature-specific).
- Pick **one** state strategy (RxJS subjects/observables in a service, or NgRx) and apply it consistently; avoid parallel ad-hoc event channels. Signals are not used.
- Always clean up subscriptions: prefer the `async` pipe in templates, or `takeUntilDestroyed()` / explicit teardown in code, to prevent leaks.

## Routing, Guards, Interceptors, Resolvers

- Define routes centrally (e.g. `app.routes.ts`). Prefer **lazy loading** for feature areas (`loadComponent` / `loadChildren`) unless the app is small.
- Protect routes with `canActivate` guards and preload data with `resolve` resolvers.
- **Guards, interceptors, and resolvers must be functional** (`CanActivateFn`, `HttpInterceptorFn`, `ResolveFn`) and obtain dependencies via `inject()`. Do not write class-based versions.
- Centralize cross-cutting HTTP behavior (auth headers, loading indicators, global error handling, `withCredentials`) in interceptors rather than scattering it across services. Provide an opt-out mechanism (e.g. a request header) instead of bypassing the interceptor chain.

## Interfaces, Models, Enums & Types

- Group related interfaces by domain in a single file under `interfaces/` (or `models/`).
- Keep shared enums and string-literal type aliases in dedicated `enums/` and `types/` locations.
- Prefer **string-literal union types** for small closed sets (e.g. `type Theme = 'light' | 'dark';`) over enums where practical.
- Type API responses explicitly; avoid leaking `any` across boundaries.

## Internationalization (i18n): standard

Every project is multi-language. Build i18n in from the start, not as an afterthought.

- **Never hardcode user-facing strings** in templates or TypeScript. All display text goes through the i18n layer with stable, namespaced keys.
- Maintain **one translation file per supported language**. Every new key must be added to **all** language files in the same change. Treat a missing key in any language as a bug (enforce with lint/CI where possible).
- Define and document the **default/fallback language**; missing translations fall back to it gracefully.
- Keep translation files in one dedicated, documented location (served as static assets or compiled, per the chosen library).
- Use **locale-aware formatting** for dates, numbers, and currency (Angular i18n pipes / the `Intl` API), not hand-rolled formatting.
- Design layouts to tolerate text expansion and (if relevant) right-to-left languages.

## Styling & Theming: light and dark are standard

Every project ships **both a light and a dark theme**. Build theming in from the start.

- Use the project's chosen preprocessor consistently (SCSS recommended); don't mix in raw CSS files if the project uses SCSS.
- **Single source of truth for design tokens.** Define colors, spacing, typography, and radii once, as **CSS custom properties** (recommended, since they enable runtime theme switching), and provide a full set of values for **both** the light and dark themes (e.g. in `styles/_theme.scss`).
- **Components reference tokens only; never hardcode colors or raw hex values.** This is what guarantees both themes work automatically. A component that reads from tokens needs no per-theme overrides.
- Apply the active theme at the root via a class or `data-theme` attribute on `:root` / `<html>`, switched at runtime. Respect the user's OS preference with `prefers-color-scheme` as the initial default, and allow an explicit user toggle that persists.
- Scope component styles with `:host` and the **direct-child combinator (`>`)**, using `&` nesting for states (`&:hover`). Component styles stay in that component's stylesheet.
- **Do not use `::ng-deep`** (or any deprecated view-piercing selector). `:host` is allowed.
- **No space between `>` and the class/element name** in selectors: `>.className`, not `> .className`.
- **SCSS variable names use `camelCase`**, not kebab-case: `$colorTextDark`, not `$color-text-dark`.
- The global stylesheet holds only global concerns (resets, base typography, keyframes, scrollbars) and the theme token definitions. Never put component-specific rules there.

## HTML / Templates

- Use the **modern control flow** (`@if`, `@for (... ; track ...)`, `@switch`), not legacy `*ngIf` / `*ngFor`. Always provide a `track` expression in `@for`.
- Use self-closing tags for content-less components (`<app-user-card ... />`).
- Don't leave blank lines between sibling elements; keep markup compact.
- Templates must pass accessibility lint rules: semantic elements, labels, alt text, keyboard support.

## TypeScript Rules

- Enable strict mode and the strict family: `strict`, `noImplicitOverride`, `noImplicitReturns`, `noFallthroughCasesInSwitch`, `noPropertyAccessFromIndexSignature`, plus Angular's `strictTemplates`. Code must compile clean under all of them.
- With `noPropertyAccessFromIndexSignature`, access index-signature/dynamic keys with **bracket notation** (`obj['key']`), not dot access.
- Keep functions and files small. Concrete limits (max lines per function/file, max line length) are configured in ESLint. **Split code when the linter flags it** rather than disabling the rule.
- Follow the project's ESLint/Prettier config exactly. Common baseline rules to expect: strict equality (`===`/`!==`), required semicolons, single quotes, no `console.log` (allow `warn`/`error`), `T[]` over `Array<T>`, no parameter reassignment, no variable shadowing. Treat the config file as authoritative over this list.

## Anti-patterns: do NOT

- Pin to an outdated Angular major without a clear reason, or use legacy APIs the current version has replaced.
- Create `NgModule`s for new code, or class-based guards/interceptors/resolvers.
- Use `*ngIf` / `*ngFor` (use `@if` / `@for`).
- Hardcode user-facing strings (always route through i18n) or add a key to only some language files.
- Hardcode colors or raw hex in components (use theme tokens; both light and dark must work).
- Hardcode API/socket URLs, import `environment.development.ts` directly, or add a config key to only some environment files.
- Use `console.log`, `==`/`!=`, or leave subscriptions un-cleaned.
- Use signals in components (use observables + explicit change detection via `ChangeDetectorRef`).
- Use `::ng-deep` (use `:host` and component-scoped styles).
- Create `*.spec.ts` files.
- Write SCSS variables in kebab-case, or put a space after `>` in selectors.
- Add new dependencies without explicit instruction.
- Duplicate an existing shared component or bypass shared cross-cutting infrastructure (interceptors, dialog/notification/loading services).

## Definition of Done

1. `ng lint` passes with no errors.
2. `ng build` (production) type-checks and succeeds. Remember production env is the default, so missing env keys fail here.
3. No new `*.spec.ts` files were created; if the project has existing tests, they still pass (`ng test`, run by the user).
4. New user-facing strings are added to **every** language file, routed through i18n (no hardcoded text).
5. New/changed UI is verified in **both light and dark themes**, using theme tokens (no hardcoded colors).
6. No hardcoded URLs, no `console.log`, no unintended new dependencies; built on a current, supported Angular version.
