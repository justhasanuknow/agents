# AGENTS.md: Entry Point for AI Agents

**Read this file first.** This is the single entry point for any AI agent (Claude Code, Cursor, GitHub Copilot, etc.) working with this repository or using its instruction files in another project. It tells you which files to read, when to read them, and in what order.

## What This Repository Is

A curated collection of standalone instruction documents for AI agents. Each file defines rules or a complete workflow for a specific task class. Files are self-contained (no shared imports), but this entry point defines which ones apply to your current task.

## Reading Order

Follow these steps in order at the start of every task:

1. Read this file (`AGENTS.md`) completely.
2. Read every file in the "Always Read" list below.
3. Determine your task type, then read the matching file(s) from the "Read When Relevant" table.
4. Only then start working.

Do not skip step 2 because the task looks small. The always-read files contain safety rules that apply to every task, including one-line changes.

## Always Read (every task, no exceptions)

| File | What it defines |
| --- | --- |
| [GENERAL-RULES.md](GENERAL-RULES.md) | Baseline rules for every task: task startup checks, coding conduct (comments explain why, no emojis, no em dashes), formatting, command/environment restrictions (no npm/node/deno, no file deletion, no long-running processes), database restrictions, and per-technology conventions. |
| [GIT-RULES.md](GIT-RULES.md) | Which git and GitHub CLI (`gh`) operations you may perform, which are strictly user-only, how conflict resolution is scoped, and how a user waiver works. Applies to every task in every repository. |

## Read When Relevant (by task type)

| If your task involves... | Read this file |
| --- | --- |
| Working in an Angular codebase (any Angular version, any task: feature, fix, refactor) | [tech-based-rules/ANGULAR.md](tech-based-rules/ANGULAR.md) |
| Building or modifying a plain static website (HTML/CSS/JS, no framework) | [tech-based-rules/PLAIN-STATIC-SITE-TEMPLATE.md](tech-based-rules/PLAIN-STATIC-SITE-TEMPLATE.md) |
| Working in a SvelteKit codebase (any task: new project, feature, fix, refactor) | [tech-based-rules/SVELTEKIT.md](tech-based-rules/SVELTEKIT.md) |
| Building a complete new application from a requirements document (`PROJECT.md`) in one autonomous run, after the user explicitly starts autonomous mode | [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) |

If no file matches your task type, proceed with the always-read rules plus your general engineering judgment. Do not force-apply an unrelated instruction file.

## Precedence

When rules appear to conflict, resolve in this order (highest wins):

1. A direct, explicit instruction from the user in the current conversation.
2. `GIT-RULES.md` and other always-read files (safety rules are never overridden by task-specific files).
3. The target project's own `AGENTS.md`. It may override technology files and style defaults, never safety rules or the indentation rule in GENERAL-RULES.md.
4. The task-specific instruction file (e.g. `ANGULAR.md`).
5. This file's general guidance.
6. Your own defaults.

If a conflict cannot be resolved with this list (for example, two files give contradictory technical directions), stop and ask the user instead of guessing.

**Autonomous mode is the one exception to item 2.** When the user explicitly starts autonomous mode as defined in [DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md), that activation is a level-1 instruction. It lifts exactly the restrictions listed in that file's "Authority" section, for the duration of one run, and nothing else. Restrictions that file lists as still in force stay in force. Without explicit activation in the current conversation, DEVELOPMENT-WORKFLOW.md grants nothing.

## Repository-Wide Conventions

These apply when you are editing this repository itself:

- **English only.** All file content, file names, and comments are written in English, regardless of the conversation language.
- **Markdown must be lint-clean.** Follow `.markdownlint.jsonc`: fenced code blocks always declare a language (MD040), table pipes are surrounded by spaces (MD060), line length is not limited (MD013 off).
- **Naming:** instruction files use SCREAMING-KEBAB-CASE (`GIT-RULES.md`); folders use kebab-case (`tech-based-rules/`).
- **Placement:** technology-specific instructions go in `tech-based-rules/`; cross-cutting rules and workflows (like git safety and the development workflow) live at the repository root.
- **Keep the catalog in sync.** When you add, rename, or move an instruction file, update the Catalog table and the Repository Structure diagram in [README.md](README.md), and the tables in this file.
- **Instructions must be unambiguous for agents.** Write rules as explicit, testable statements. Prefer "never do X; do Y instead" over vague guidance. Include decision procedures for edge cases.
- **Line endings are LF**, enforced by `.gitattributes`.
- **`.markdownlint.jsonc` is the authoritative lint configuration.** The rules described in this list must match it.
- **Prose follows GENERAL-RULES.md**: no emojis, and no em dashes, en dashes or double hyphens as punctuation.

## What This File Is Not

- It is not a task instruction itself. It only routes you to the right one.
- It does not replace the user's judgment: `README.md` explains the repository to humans; this file instructs agents. If you change the structure of the repository, keep both in sync.
