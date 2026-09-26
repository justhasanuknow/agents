# GENERAL-RULES.md: Baseline Rules for AI Agents

These rules apply to **every task in every project**, regardless of technology. Read this file together with [AGENTS.md](AGENTS.md) and [GIT-RULES.md](GIT-RULES.md) before starting any work.

Technology-specific instruction files (in `tech-based-rules/`) build on top of this baseline:

- **Safety restrictions** in this file (file deletion, database access, command restrictions, git) can **never** be overridden by a technology-specific file.
- **Style conventions** in this file are defaults. If a technology-specific instruction file defines a more specific convention for the same topic, the more specific rule wins.
- **Autonomous mode** ([DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md)), when the user explicitly starts it, lifts the command, file-deletion, database and git restrictions in this file within the scope that file defines. It is not a technology-specific file, and it is the only document that can do this.

---

## Task Startup

Do these at the beginning of every task, before writing anything:

1. **Check whether the target repository contains an `AGENTS.md` file.** If it exists, read it and follow it.
2. **Scan the repository.** Understand its structure, tooling, and conventions before making changes.
3. **Study similar files first.** Before creating or modifying a file, read existing files of the same kind and follow their patterns, naming, and conventions. Preserve the codebase's structural integrity: your changes should be indistinguishable in style from the surrounding code.

---

## General Conduct

- **Comments explain why, never what.** Never add comments that restate what the code does, TODO markers or commented-out code. Comments are allowed, in English, only for: the reason behind a non-obvious decision, a workaround and what it works around, and documentation comments (JSDoc/TSDoc) on exported functions, types and components.
- **Never use emojis**: not in code, file names, commit messages, or generated documents.
- **Never use em dashes (—), en dashes (–) or double hyphens (--) as punctuation** in any text you write: documents, code comments, commit messages, UI copy, reports and replies. Use a comma, a colon, parentheses or a new sentence instead. Numeric ranges use a plain hyphen (`3-5`). This does not apply where the characters are code syntax: command-line flags (`--force`), Markdown horizontal rules and table separators (`---`), operators and similar.
- **Follow the existing codebase's standards.** Every codebase has established writing styles; match them exactly. This applies with particular care to SCSS files.

---

## Linting & Formatting

- **Prettier is authoritative for formatting**, except indentation. Follow the project's `.prettierrc` exactly; never introduce formatting that Prettier would rewrite.
- **Indentation is always tabs, with a tab width of 4.** No technology file, project file or formatter config overrides this. In a new project, configure the formatter and `.editorconfig` for it (Prettier: `"useTabs": true, "tabWidth": 4`). If an existing project's config sets different indentation, do not start editing: tell the user, give them the exact config change, and continue once it is applied.
- **Opening braces on the same line** as the function/statement declaration, never on a new line.
- **Keep function bodies under 150 lines** where practical; split longer functions into smaller pieces. This limit is flexible: if a function cannot be split cleanly, ask the user before exceeding it.
- **Never use the ternary operator (`?:`) for if/else logic** in TypeScript or JavaScript. Write explicit `if`/`else` statements instead. The only exception is inline value selection inside template expressions, where a technology file explicitly allows it ([SVELTEKIT.md](tech-based-rules/SVELTEKIT.md) does).

---

## Command & Environment Restrictions

- **Do not run `npm` or `node` commands.** If one is needed, tell the user the exact command and ask them to run it.
- **Do not run `deno` commands.** Same protocol: give the user the exact command to run.
- **Never delete any file directly.** If a file no longer serves a purpose, explain to the user why that is the case and ask the user to delete it themselves.
- **Never start long-running processes**: dev servers, watch modes, daemons, or anything that does not terminate on its own.
- **Exception:** inside an explicitly started autonomous run, [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) defines what is allowed.

---

## Git

Full rules live in [GIT-RULES.md](GIT-RULES.md). Read that file for the complete operation lists and procedures. The core of it:

- **Never perform git actions** (including `git add`, `git commit`, `git push`). Read-only inspection commands (`git status`, `git log`, `git diff`, etc.) are allowed.
- **The same applies to the GitHub CLI.** `gh` commands that read (`gh repo view`, `gh pr list`, `gh api` without a method) are allowed; anything that writes to a remote (releases, pull requests, issues, repository settings, secrets, workflow runs) is user-only.
- If the user requests a git or `gh` action, **inform them of this rule and give them the exact command to run themselves**.
- **Exception:** inside an explicitly started autonomous run, [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) defines what is allowed.

---

## Database

- **Never directly perform any database operation**: no queries, migrations, schema changes, or data modifications against any database.
- If a database operation is needed, **provide the user with everything required to run it themselves**: the exact SQL commands, the order to run them in, and any warnings about destructive effects.
- **Exception:** inside an explicitly started autonomous run, [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) defines what is allowed.

---

## Frontend

### General (framework-agnostic)

- **TypeScript: blank lines between logically distinct blocks.** When consecutive expressions change subject, separate them with a blank line. Example: statements updating `tr` language values and statements updating `en` language values are two subjects. Put a blank line between the two groups.
- **HTML: no blank lines between tags.** If a `<div>` closes on line 56, the next sibling `<div>` starts on line 57, not 58. Keep markup compact.
- **Never use the `any` type.** Type everything explicitly. Define interfaces in separate files:
  - Used by only one file → create the interface file next to that file.
  - Shared across files → create it in the `modules/interfaces/` folder (create the folder if it does not exist), unless the technology-specific instruction file defines a different location for that stack.

### SCSS

- **Variable names use `camelCase`, not `kebab-case`.** Example: `$colorTextDark`, not `$color-text-dark`.
- **No space between `>` and the class/element name.** Example: `>.className`, not `> .className`.
- **Use the direct child combinator (`>`)** in selectors when targeting direct children.

### Angular

Full conventions live in [tech-based-rules/ANGULAR.md](tech-based-rules/ANGULAR.md). Read it for any Angular task. Headline rules:

- Do not use `::ng-deep`. `:host` is allowed.
- Do not use signals when creating components. Use observables and manage change detection explicitly (inject `ChangeDetectorRef` if needed).
- Do not create `*.spec.ts` files.
- Always use standalone components.

### SvelteKit

Full conventions live in [tech-based-rules/SVELTEKIT.md](tech-based-rules/SVELTEKIT.md). Read it for any SvelteKit task. Headline rules:

- Svelte 5 runes only; no legacy syntax (`export let`, `$:`, `on:event`, `<slot>`).
- Server-only code lives in `src/lib/server/`.
- Drizzle ORM is the only database access layer.
- SCSS for styles, CSS custom properties for light and dark theming.
- Paraglide JS for i18n; no hardcoded user-facing strings.
