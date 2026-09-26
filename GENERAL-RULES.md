# GENERAL-RULES.md — Baseline Rules for AI Agents

These rules apply to **every task in every project**, regardless of technology. Read this file together with [AGENTS.md](AGENTS.md) and [GIT-RULES.md](GIT-RULES.md) before starting any work.

Technology-specific instruction files (in `tech-based-rules/`) build on top of this baseline:

- **Safety restrictions** in this file (file deletion, database access, command restrictions, git) can **never** be overridden by a technology-specific file.
- **Style conventions** in this file are defaults — if a technology-specific instruction file defines a more specific convention for the same topic, the more specific rule wins.
- **Autonomous mode** ([DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md)), when the user explicitly starts it, lifts the command, file-deletion, database and git restrictions in this file within the scope that file defines. It is not a technology-specific file, and it is the only document that can do this.

---

## Task Startup

Do these at the beginning of every task, before writing anything:

1. **Check whether the target repository contains an `AGENTS.md` file.** If it exists, read it and follow it.
2. **Scan the repository.** Understand its structure, tooling, and conventions before making changes.
3. **Study similar files first.** Before creating or modifying a file, read existing files of the same kind and follow their patterns, naming, and conventions. Preserve the codebase's structural integrity — your changes should be indistinguishable in style from the surrounding code.

---

## General Conduct

- **Never add comments** to code. No explanatory comments, no TODO markers, no commented-out code.
- **Never use emojis** — not in code, file names, commit messages, or generated documents.
- **Follow the existing codebase's standards.** Every codebase has established writing styles; match them exactly. This applies with particular care to SCSS files.

---

## Linting & Formatting

- **Prettier is authoritative.** Follow the rules defined in the project's `.prettierrc` file exactly; never introduce formatting that Prettier would rewrite.
- **Indentation: tabs, with a tab width of 4.**
- **Opening braces on the same line** as the function/statement declaration, never on a new line.
- **Keep function bodies under 150 lines** where practical; split longer functions into smaller pieces. This limit is flexible — if a function cannot be split cleanly, ask the user before exceeding it.
- **Never use the shorthand ternary (`?`) operator for if/else logic.** Write explicit `if`/`else` statements instead.

---

## Command & Environment Restrictions

- **No `npm` or `node` commands.** You do not have access to run them. If one is needed, tell the user the exact command and ask them to run it.
- **No `deno` commands.** Same protocol: give the user the exact command to run.
- **Never delete any file directly.** If a file no longer serves a purpose, explain to the user why that is the case and ask the user to delete it themselves.
- **Never start long-running processes** — dev servers, watch modes, daemons, or anything that does not terminate on its own.
- **Exception:** inside an explicitly started autonomous run, [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) defines what is allowed.

---

## Git

Full rules live in [GIT-RULES.md](GIT-RULES.md) — read that file for the complete operation lists and procedures. The core of it:

- **Never perform git actions** (including `git add`, `git commit`, `git push`). Read-only inspection commands (`git status`, `git log`, `git diff`, etc.) are allowed.
- **The same applies to the GitHub CLI.** `gh` commands that read (`gh repo view`, `gh pr list`, `gh api` without a method) are allowed; anything that writes to a remote — releases, pull requests, issues, repository settings, secrets, workflow runs — is user-only.
- If the user requests a git or `gh` action, **inform them of this rule and give them the exact command to run themselves**.
- **Exception:** inside an explicitly started autonomous run, [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) defines what is allowed.

---

## Database

- **Never directly perform any database operation** — no queries, migrations, schema changes, or data modifications against any database.
- If a database operation is needed, **provide the user with everything required to run it themselves**: the exact SQL commands, the order to run them in, and any warnings about destructive effects.
- **Exception:** inside an explicitly started autonomous run, [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) defines what is allowed.

---

## Frontend

### General (framework-agnostic)

- **TypeScript — blank lines between logically distinct blocks.** When consecutive expressions change subject, separate them with a blank line. Example: statements updating `tr` language values and statements updating `en` language values are two subjects — put a blank line between the two groups.
- **HTML — no blank lines between tags.** If a `<div>` closes on line 56, the next sibling `<div>` starts on line 57, not 58. Keep markup compact.
- **Never use the `any` type.** Type everything explicitly. Define interfaces in separate files:
  - Used by only one file → create the interface file next to that file.
  - Shared across files → create it in the `modules/interfaces/` folder (create the folder if it does not exist), unless the technology-specific instruction file defines a different location for that stack.

### SCSS

- **Variable names use `camelCase`, not `kebab-case`.** Example: `$colorTextDark`, not `$color-text-dark`.
- **No space between `>` and the class/element name.** Example: `>.className`, not `> .className`.
- **Use the direct child combinator (`>`)** in selectors when targeting direct children.

### Angular

Full conventions live in [tech-based-rules/ANGULAR.md](tech-based-rules/ANGULAR.md) — read it for any Angular task. Headline rules:

- Do not use `::ng-deep`. `:host` is allowed.
- Do not use signals when creating components. Use observables and manage change detection explicitly (inject `ChangeDetectorRef` if needed).
- Do not create `*.spec.ts` files.
- Always use standalone components.

### SvelteKit

No rules defined yet.

### React

No rules defined yet.

---

## Backend

### Node.js

No rules defined yet.

### Deno

No rules defined yet.

### Supabase

No rules defined yet.

---

## Mobile

### Flutter

- **Never add or update dependencies in `pubspec.yaml`** without explicit user approval.
- **When creating a new `*_service.dart` file in `lib/core/services/`**, immediately update `PROJECT.MD` §2 (Servisler) with the new service's method signatures.
