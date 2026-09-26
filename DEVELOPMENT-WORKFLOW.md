# DEVELOPMENT-WORKFLOW.md: Autonomous One-Shot Project Development

This workflow builds a complete new application from a requirements document in one uninterrupted run. The agent analyzes the requirements, splits the work into phases, gates every phase with tests that include all earlier phases' tests, commits and pushes each finished phase to a private repository it creates, and finishes with a maintainer guide. It does not stop for approval between steps.

Read this file together with [AGENTS.md](AGENTS.md), [GENERAL-RULES.md](GENERAL-RULES.md) and [GIT-RULES.md](GIT-RULES.md). It does not replace them. It lifts a defined set of their restrictions for the duration of one run, and nothing else.

---

## Activation

Autonomous mode is off by default. It starts only when the user, in the current conversation, explicitly instructs the agent to follow this file in autonomous mode. Example:

```text
Follow DEVELOPMENT-WORKFLOW.md in autonomous mode. The requirements are in PROJECT.md.
```

- Attaching this file, mentioning it, or saying "build this" is not activation. Without explicit activation, every rule in GENERAL-RULES.md and GIT-RULES.md applies unchanged.
- Activation covers one run of one project in the current working directory. A run ends when the final report is delivered, or when the run stops at a blocked gate.
- Resuming an interrupted run (see "State and Resumption") requires the same explicit instruction again.
- Activation expires with the run. It never carries over to other tasks, other directories or later conversations.

---

## Scope

This workflow is only for a **new** application built from scratch.

- `PROJECT.md` must exist in the working directory. It is the source of truth for what to build. The agent never modifies it.
- Apart from `PROJECT.md` and the files it references (assets, samples, specifications), the working directory must be empty. If it contains an existing codebase or a `.git` directory, and no run state exists (see "State and Resumption"), this workflow does not apply: stop and tell the user.

Changes to an existing codebase, however large, are normal tasks under the normal rules.

---

## Authority

### Lifted during the run

| Area | Allowed |
| --- | --- |
| Package managers and runtimes | Run `npm`, `pnpm`, `node`, `deno`, `pip`, `cargo` and equivalent tools; install the project's dependencies into the project. |
| Long-running processes | Start dev servers, watchers, test runners and anything else the build or the tests need. |
| Docker | Build images; create, start, stop and remove containers, volumes and networks that this run created. Prefix every one of them with the project's slug. |
| Databases | Any operation (migrations, seeds, queries, resets) against databases running in containers this run created. |
| Files | Create, modify and delete files inside the project directory. |
| Git | `git init`, `git add`, `git commit`, creating local branches, and non-force `git push` to this run's own repository. |
| GitHub CLI | `gh repo create <name> --private`, once, for this project, set as `origin`. |

Anything not listed in this table remains forbidden.

### Still in force during the run

| Rule | Detail |
| --- | --- |
| No agent attribution | The Commit Message Rules in GIT-RULES.md apply in full: no AI `Co-Authored-By` trailers, no "Generated with" footers, no AI markers of any kind. |
| Conduct and style | The General Conduct and Linting & Formatting rules in GENERAL-RULES.md and the matching technology file apply. All repository content is in English. |
| Private only | Never create a public repository. Never change visibility; never rename, archive, transfer or delete a repository; never edit repository settings. |
| No history rewrite | No force-push, `rebase`, `reset --hard`, amending pushed commits or `filter-repo`. Never delete remote branches or tags. |
| Other GitHub writes | No releases, issues, pull requests, comments, secrets, variables, workflow runs, gists or `gh auth` changes. |
| Outside the project | Never touch files outside the project directory. Never touch containers, volumes, networks or databases this run did not create. Never change global or system configuration: `git config`, global package installs, shell profiles, system packages. |
| No deployment | Never deploy, never connect to production or remote servers, never change DNS. Deployment files are written and documented, never executed against real infrastructure. |
| External services | Tests never call real third-party services; use mocks, fakes or local containers. Real credentials are used only if PROJECT.md explicitly provides them for that purpose. |
| Secrets | Never commit secrets or `.env` files. Commit a `.env.example` that documents every variable. |

### Precedence within the run

- Technology-specific files matching the project's stack apply (see the table in AGENTS.md).
- Tests are mandatory in a run. Where a technology file forbids test files (for example the `*.spec.ts` rule in ANGULAR.md), this workflow wins for the duration of the run.
- PROJECT.md decides what to build (stack, features, extra guides) and overrides this file's defaults on those points. It can never widen the Authority above.

---

## Project Files

| File | Owner | Purpose |
| --- | --- | --- |
| `PROJECT.md` | User | Requirements. Never modified by the agent. |
| `docs/ANALYSIS.md` | Agent | Requirements inventory, constraints, size and complexity assessment. |
| `docs/PLAN.md` | Agent | Phases, test plans, traceability, status. The run's state. |
| `docs/DECISIONS.md` | Agent | Every decision made without the user. |
| `docs/MANUAL-CHECKS.md` | Agent | Checks that cannot be automated, if any. |
| `docs/OPERATIONS.md` | Agent | Maintainer guide. |
| `docs/RUN-REPORT.md` | Agent | Final report of the run. |
| `CHANGELOG.md` | Agent | Changes per phase. |
| `README.md` | Agent | Short overview, quick start, links into `docs/`. |
| `.env.example` | Agent | Every environment variable, documented. |

---

## Workflow

### Step 0: Preflight

Before writing anything:

1. Read AGENTS.md and the files it requires, then read PROJECT.md completely.
2. If `docs/PLAN.md` exists, this is a resume: go to "State and Resumption".
3. Check the working directory against "Scope".
4. Check the tools: the runtimes and services the project needs, Docker if the project needs it, `git` with a user identity already configured (`git config --get user.name` and `user.email`), and `gh auth status` authenticated.
5. Determine the repository name: from PROJECT.md if given, otherwise the kebab-case project name. Check with `gh repo view`; if a repository with that name already exists, stop.

If any preflight check fails, stop and tell the user exactly what is missing. Preflight and a blocked gate are the only two points where a run stops.

### Step 1: Analysis

Write `docs/ANALYSIS.md`:

- Every requirement in PROJECT.md with an ID (`REQ-001`, `REQ-002`, ...), functional and non-functional: security, performance, internationalization, accessibility, deployment target.
- Constraints: stack, versions, hosting target, integrations.
- What PROJECT.md explicitly puts out of scope.
- Every gap, ambiguity and contradiction, each resolved through the Decision Policy.
- A size and complexity assessment (modules, integrations, data model, risk areas) that concludes with the number of phases and why.

### Step 2: Plan

Write `docs/PLAN.md`. It is not submitted for approval; the run continues immediately.

- **Phase 1** is always: repository creation, project scaffold, tooling (formatter, linter, type checking), test infrastructure with one passing test, `.env.example`, and the Docker setup if the project uses Docker.
- Phases are ordered by dependency. Every phase ends in a working, buildable state.
- A phase is a coherent slice that can be tested completely. If a phase would cover unrelated modules, split it. There is no fixed phase count; a small project may have two or three.
- Each phase states: number and title, goal, requirement IDs covered, deliverables, test plan, acceptance criteria and status (`pending`, `in-progress`, `done`, `blocked`).
- Each test in a test plan states what it verifies, its type (unit, integration, end-to-end) and the requirement IDs it covers.
- Test plans are comprehensive: happy paths, validation and error paths, authorization boundaries, and every edge case PROJECT.md names. If the project has a UI, every user-facing flow has an end-to-end test.
- The plan ends with a traceability table: requirement ID, phase, tests. Every requirement maps to at least one phase and at least one test.

### Step 3: Phase Loop

For each phase, in order:

1. Set its status to `in-progress` in `docs/PLAN.md`.
2. Implement the deliverables.
3. Write the planned tests. Tests live in the repository, and the full suite runs with a single command documented in README.md.
4. Run the gate.
5. Gate green: update `CHANGELOG.md`, set the status to `done` with the test totals, commit, push, and start the next phase without stopping.
6. Gate red: see "Failure Handling".

Intermediate commits and pushes during a phase are allowed. A phase-completing commit is only made on a green gate.

#### The gate

A phase passes only when all of the following hold in a single run of the full suite:

- Every test of the current phase passes.
- Every test of every earlier phase passes. The full suite always runs, never a subset.
- The build succeeds, and lint and type checks pass if the stack has them.
- Every entry in `docs/MANUAL-CHECKS.md` for the current and earlier phases is performed again and its result recorded.
- No test is skipped, pending, focused (`.only`), disabled or marked as expected to fail.

A test that passes only on retry is a failing test. Fix the cause of the flakiness; never add retries to hide it.

#### Test integrity

- Never delete, skip, disable or weaken a test to make the gate pass. Weakening includes loosening an assertion, widening a tolerance, and mocking away the behavior under test.
- An earlier phase's test may change only when a later requirement deliberately changes the behavior it verifies. Record every such change in `docs/DECISIONS.md` with the requirement ID, the old expectation and the new one.
- `docs/MANUAL-CHECKS.md` is only for checks that genuinely cannot be automated. Each entry states why, the steps and the expected result. The agent performs them itself (for example through browser automation or HTTP requests) at every gate and records the result.

#### Failure handling

- An attempt is one fix followed by one full gate run.
- A phase gets at most **5 attempts** on a red gate.
- After the fifth failed attempt, the run stops:
  1. Create the branch `blocked/phase-<n>`, commit the current work there and push it. `main` keeps the last green phase.
  2. Set the phase status to `blocked` in `docs/PLAN.md`, with the failing tests, what was tried and the best current hypothesis.
  3. Stop every process and container this run started.
  4. Deliver the final report.
- Never start the next phase while the gate is red.

### Step 4: Delivery

After the last phase is `done`:

1. Run the full gate once more from a clean state: fresh dependency install, freshly created containers.
2. Write `docs/OPERATIONS.md` (see "Maintainer Guide").
3. Write an end-user guide only if PROJECT.md asks for one, in the form and location it specifies.
4. Write `README.md`: what the project is, quick start, the test command, links into `docs/`.
5. Update `CHANGELOG.md`, write `docs/RUN-REPORT.md`, commit and push.
6. Stop every process and container this run started. Leave images and volumes in place and list them in the report.
7. Deliver the final report in the conversation.

---

## Maintainer Guide

`docs/OPERATIONS.md` is written for the person who installs, deploys and maintains the application, not for end users. Someone who has never seen the project must be able to run and maintain it using only this guide. It contains:

1. **Overview**: purpose, architecture, components and how they communicate.
2. **Modules and features**: for each module, its responsibility, its location in the code, its main features and the configuration it reads.
3. **Requirements**: runtimes and versions, external services, ports.
4. **Configuration**: every environment variable with purpose, required or optional, default and example.
5. **Local setup**: exact commands from clone to running application.
6. **Docker**: building, running, compose services, volumes and what each one holds.
7. **Deployment**: step by step for the target in PROJECT.md (a generic Docker host if none is given), reverse proxy and TLS notes, first-run bootstrap such as creating the first admin.
8. **Database**: migrations, seeding, backup and restore.
9. **Testing**: running the full suite and each test type; where the tests live.
10. **Updating**: dependency updates, applying new versions and migrations safely.
11. **Operations**: logs, health checks, common failures and their fixes.
12. **Security**: secret handling and rotation, summary of roles and permissions.
13. **Known limitations** and deviations from PROJECT.md.

Every command in the guide must be one the agent ran or verified during the run. Deployment steps against real infrastructure cannot be verified; mark them as unverified.

---

## Decision Policy

The run never stops to ask a question. When PROJECT.md is silent, ambiguous or contradictory:

1. Choose the option most consistent with PROJECT.md's stated goals and constraints.
2. Between otherwise equal options, prefer the more reversible, the simpler and the better established.
3. Never expand scope: do not add features, roles or integrations PROJECT.md does not describe. Implementation details (a library, an internal module) are not scope.
4. New dependencies must be widely used, actively maintained and compatibly licensed.
5. Record every decision in `docs/DECISIONS.md`: ID (`DEC-001`, ...), context, options considered, choice, rationale, affected requirements and phases, and `review: yes` when the user should look at it.

If an input only the user can provide is missing (a credential, an asset, a business rule) and a requirement cannot be completed without it, implement everything else, put the missing part behind a clear interface or stub, record it with `review: yes`, and list it in the final report. It is not a reason to stop.

---

## State and Resumption

The run must survive context compaction, a crashed session and a restart.

- `docs/PLAN.md` is the single source of run state. Update it on every status change, before committing.
- Never rely on memory of earlier parts of the conversation. After any context compaction, re-read this file, PROJECT.md, `docs/PLAN.md` and `docs/DECISIONS.md` before continuing.
- On resume (the user re-activates autonomous mode in a directory where `docs/PLAN.md` exists):
  1. Read the files above, then `git status` and `git log`.
  2. Run the full gate to confirm the last `done` phase is still green. If it is not, fix it first under "Failure Handling".
  3. Continue with the first phase that is not `done`.
  4. For a `blocked` phase, follow the instructions in the user's resume message first. Its attempt count starts again from zero.

---

## Commit Conventions

- Default branch: `main`.
- Phase-completing commit subject: `Phase <n>: <phase title>`. Body: the requirement IDs covered, test totals (passed / total), and decision IDs added in the phase.
- Other commits: a concise imperative subject line.
- Push after every phase-completing commit.
- No agent attribution (see "Authority").

---

## Final Report

Delivered in the conversation and saved as `docs/RUN-REPORT.md`:

- Outcome: completed, or blocked (which phase and why).
- Repository URL and final commit hash.
- Every phase with status, requirement IDs and test totals.
- Full-suite totals from the last gate.
- Every decision marked `review: yes`.
- Deviations from PROJECT.md and stubbed parts.
- Known limitations.
- How to run the project, or a link to `docs/OPERATIONS.md`.
- Images and volumes left on the machine. Processes and containers should all be stopped; list any that are not.

Every number and status in the report comes from actual command output, never from assumption.

---

## Validation Checklist

Before delivering the final report:

- [ ] Every requirement maps to a phase and a test; the traceability table is complete.
- [ ] Every phase is `done`, or the run stopped with a documented `blocked` phase.
- [ ] The final gate is green from a clean state, with zero skipped tests.
- [ ] `docs/OPERATIONS.md` is complete and its commands were verified.
- [ ] No secret exists anywhere in the git history; `.env.example` is complete.
- [ ] No commit contains agent attribution.
- [ ] The repository is private (`gh repo view --json visibility`).
- [ ] Every process and container the run started is stopped.
- [ ] Nothing outside the project directory was touched.
