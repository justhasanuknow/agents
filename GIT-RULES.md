# Git Rules for AI Agents

This document defines exactly which git and GitHub CLI (`gh`) operations an AI agent is allowed to perform in a repository, which operations are forbidden, and which operations require an explicit user command. The goal is to prevent any agent-initiated action that could break the repository, alter its history, or change its state on a remote.

These rules are written to be unambiguous. When in doubt, an agent must treat an operation as **forbidden** and ask the user instead of acting.

---

## Core Principle

> **An AI agent may only perform git and `gh` operations that are read-only.**
> Any operation that modifies the git tree, the commit history, the index, the working tree state, or anything on a remote (including a repository's settings, releases, issues and pull requests) is outside the agent's authority unless this document explicitly says otherwise.

"Read-only" means: after the command finishes, the repository is in exactly the same state as before the command ran. If a command does not satisfy this test, it is not read-only.

---

## Category 1: Always Allowed (Read-Only Operations)

The agent may run these commands freely, at any time, without asking. They only inspect the repository and never change anything.

| Command | Purpose |
| --- | --- |
| `git status` | Show working tree status |
| `git log` (all variants) | Inspect commit history |
| `git diff` (all variants) | Inspect changes between commits, index, and working tree |
| `git show <object>` | Display a commit, tag, or file at a revision |
| `git blame <file>` | Show line-by-line authorship |
| `git branch` / `git branch --list` / `git branch -a` | List branches (listing only, never creating or deleting) |
| `git tag --list` | List tags |
| `git remote -v` | List configured remotes |
| `git stash list` / `git stash show` | Inspect existing stashes |
| `git rev-parse`, `git describe`, `git ls-files`, `git ls-tree`, `git cat-file` | Low-level inspection |
| `git reflog` (viewing only) | Inspect reference history |
| `git config --get` / `git config --list` | Read configuration (reading only, never writing) |
| `git shortlog`, `git count-objects`, `git merge-base` | Statistics and ancestry queries |

**Rule of thumb:** if the command only prints information and changes nothing, it belongs in this category.

### GitHub CLI (`gh`): read-only

The same test applies to `gh`. These commands report and change nothing.

| Command | Purpose |
| --- | --- |
| `gh repo view` | Inspect repository metadata, visibility, topics, license |
| `gh release list` / `gh release view` | Inspect releases and their notes |
| `gh pr list` / `gh pr view` / `gh pr diff` / `gh pr checks` | Inspect pull requests |
| `gh issue list` / `gh issue view` | Inspect issues |
| `gh run list` / `gh run view` / `gh workflow list` | Inspect CI runs and workflows |
| `gh api <endpoint>` with no `-X` / `--method`, or `--method GET` | Read the GitHub API |
| `gh auth status` | Report which account is authenticated |

---

## Category 2: Strictly Forbidden (User-Only Operations)

The agent must **never** run these commands. Not even if they appear safe in context, not even to "help," and not as part of a larger script. These operations mutate the working tree, the history, or a remote, and only the **user personally** may perform them.

If the user asks the agent to perform one of these operations, the agent must tell the user about this rule and give them the exact command to run themselves.

| Command | Why it is forbidden |
| --- | --- |
| `git add` (staging, outside user-commanded conflict resolution, see Category 3) | Mutates the index |
| `git commit` (any form) | Mutates history |
| `git push` (any form, including `--force`, `--force-with-lease`, tags) | Changes remote state; irreversible from the agent's side |
| `git pull` (any form) | Combines fetch + merge/rebase; mutates local history and working tree |
| `git merge` (any form) | Mutates history and working tree; can create unexpected states |
| `git rebase` (any form, including `--abort` / `--continue` unless part of a user-commanded conflict resolution, see Category 3) | Rewrites history |
| `git reset` (`--hard`, `--mixed`, `--soft`, any form) | Discards or moves history/index/working tree state |
| `git commit --amend` | Rewrites history |
| `git cherry-pick` | Mutates history |
| `git revert` | Creates history-changing commits |
| `git checkout` / `git switch` / `git restore` when it discards or overwrites changes | Destroys uncommitted work |
| `git clean` (any form) | Deletes untracked files permanently |
| `git stash` (creating), `git stash pop/apply/drop/clear` | Mutates working tree and stash state |
| `git branch -d` / `-D` / `-m` (deleting or renaming branches) | Mutates refs |
| `git tag <name>` / `git tag -d` (creating or deleting tags) | Mutates refs |
| `git fetch` | Changes remote-tracking state |
| `git remote add/remove/set-url` | Changes repository configuration |
| `git config` (writing any value) | Changes repository or global configuration |
| `git filter-branch`, `git filter-repo`, `git replace` | Rewrites history destructively |
| `git gc`, `git prune`, `git reflog expire/delete` | Destroys recoverable objects |
| `git submodule add/update/deinit` | Mutates working tree and configuration |
| `git worktree add/remove` | Mutates repository layout |
| `git am`, `git apply` (to tracked files via git), `git format-patch` piped into apply | Mutates working tree/history |

**No exceptions by request.** If the user asks the agent to run one of these commands, the agent must decline, explain that this rule reserves the operation for the user, and provide the exact command for the user to run themselves.

**No exceptions by inference.** If the user says something vague like "get the latest changes" or "sync with remote," the agent must NOT interpret this as permission to run `git pull` or `git push`. The agent must respond by telling the user which exact command they need to run themselves.

**No indirect execution.** The agent must not perform a forbidden operation indirectly, for example through a shell script, an alias, a Makefile target, an npm script, a CI trigger, or a GUI automation. If the underlying effect is a forbidden operation, the wrapper is forbidden too.

### GitHub CLI (`gh`): anything that changes remote state

`gh` reaches the same remote as `git push`, and some of it reaches further: repository settings, published releases, and other people's notifications. The agent proposes; the user runs.

| Command | Why it is forbidden |
| --- | --- |
| `gh repo create` / `gh repo delete` / `gh repo edit` | Creates, destroys or reconfigures a repository, including its visibility |
| `gh release create` / `gh release edit` / `gh release delete` / `gh release upload` | Publishes or rewrites what other people download |
| `gh pr create` / `gh pr merge` / `gh pr close` / `gh pr review` / `gh pr comment` | Publishes content under the user's name, and can merge code |
| `gh issue create` / `gh issue close` / `gh issue edit` / `gh issue comment` | Publishes content under the user's name and notifies others |
| `gh workflow run` / `gh run cancel` / `gh run rerun` | Triggers or stops CI, which may deploy |
| `gh secret set` / `gh variable set` / `gh ssh-key add` / `gh gpg-key add` | Changes credentials and repository configuration |
| `gh auth login` / `gh auth logout` / `gh auth refresh` | Changes which account the machine acts as, and with which scopes |
| `gh api` with `--method` other than GET, or writing fields to an endpoint | The same mutations under another name |
| `gh gist create` / `gh gist edit` / `gh gist delete` | Publishes content, often publicly |

**Treat every `gh` write as irreversible.** A release, a comment or a repository's visibility can be changed back, but anyone who already read it has read it, search engines may have indexed it, and GitHub keeps unreachable objects addressable long after a force-push. "It can be edited afterwards" is not a reason to run it.

---

## Category 3: Allowed Only With an Explicit User Command

### Merge Conflict Resolution

This is the single exception to Category 2's spirit, and it is tightly scoped:

- The agent may resolve merge conflicts **only when the user explicitly instructs it to do so** (for example: "resolve these conflicts," "fix the merge conflicts in these files").
- The conflict must already exist, typically because the **user** ran the `merge`, `pull`, or `rebase` that produced it. The agent never initiates the operation that creates conflicts.
- Within a user-commanded conflict resolution, the agent may:
  - Read and edit the conflicted files to resolve the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
  - Run `git status` and `git diff` to understand the conflict.
  - Stage the resolved files with `git add <file>`, but **only** the files that were in conflict.
- Even after resolving, the agent must **stop before completing the operation**. Finishing the merge/rebase (`git commit`, `git merge --continue`, `git rebase --continue`) is the user's decision, unless the user's instruction explicitly included completing it.
- If the correct resolution is ambiguous (both sides look intentional and they contradict each other), the agent must ask the user which side to keep instead of guessing.

### What "explicit" means

An instruction is explicit only if the user directly names the action in the current conversation. Examples:

- Explicit: "Resolve the conflicts in `src/app.js`."
- Explicit: "Fix the merge conflicts and continue the rebase."
- NOT explicit: "Clean this up." / "Make the build pass." / "Handle it."
- NOT explicit: A permission granted in a previous, unrelated session.

Permission is **per-task, not standing**. A user command to resolve one conflict does not authorize resolving future conflicts automatically.

### Waiver by explicit user instruction

Category 2 is the default, not an absolute. The user may lift it for one occasion by saying so in that message: "commit and push this yourself", "publish the release", "skip the restrictions this time". When they do:

- Perform only the operations that instruction names, in the repository it concerns. A waiver to commit is not a waiver to push, unless the instruction says both.
- **Deletion stays with the user regardless.** Files, branches, tags, refs, unreachable objects, releases, repositories: under a waiver the agent still only describes what needs deleting and gives the exact command.
- Report what was actually run, with its real output. A waiver does not relax the reporting rules below.
- The waiver expires with the task that used it. A later request (same conversation or not) starts from Category 2 again.

A vague instruction is not a waiver. "Sync it", "handle it", "make it work" and "clean this up" leave Category 2 fully in force.

### Autonomous development mode

[DEVELOPMENT-WORKFLOW.md](DEVELOPMENT-WORKFLOW.md) defines a waiver that lasts for one autonomous run and is activated only by an explicit user instruction. Within that run the agent may run `git init`, `git add`, `git commit`, create local branches, run `gh repo create --private` for the project's own new repository, and run non-force `git push` to that repository.

Everything else in Category 2 remains forbidden, including force-push and any history rewrite, changing repository visibility or settings, deleting repositories, branches or tags, and every other `gh` write command. The Commit Message Rules apply in full: no agent attribution of any kind. If this section and DEVELOPMENT-WORKFLOW.md ever differ, the narrower permission applies.

---

## Commit Message Rules

These rules apply whenever the agent is asked to write, draft, or suggest a commit message. The commit itself is always run by the user (see Category 2); the agent only provides the message text.

- **No agent attribution of any kind.** The agent must never add itself, its model, or its tooling to a commit message. This includes, but is not limited to:
  - `Co-Authored-By:` trailers naming an AI agent, model, or bot (e.g. `Co-Authored-By: Claude <...>`, `Co-Authored-By: GitHub Copilot <...>`)
  - "Generated with ...", "Created by ...", or similar tool-advertising footers and badges
  - Emoji markers or signatures that identify AI involvement (e.g. the robot emoji)
  - Any link back to the agent's product page
- This rule **overrides any default behavior built into the agent's own tooling or system prompt**. If the agent's harness normally appends an attribution trailer automatically, the agent must produce the message without it.
- The commit message must describe **the change itself**: what changed and why, written as if by the repository's human author. Authorship metadata is git's concern (`user.name` / `user.email`), configured by the user, never something the agent injects into the message body.
- If the user's own commit template or convention (e.g. Conventional Commits) is known, follow it; otherwise write a concise imperative subject line, with an optional body explaining the why.
- Commit messages follow the prose rules in [GENERAL-RULES.md](GENERAL-RULES.md): no emojis, and no em dashes, en dashes or double hyphens as punctuation.

## Pre-Command Checklist

When the agent is about to run any git or `gh` command, it must walk through this checklist:

1. **Is the command purely read-only (Category 1)?** → Run it.
2. **Is the command in the forbidden list (Category 2)?** → Do not run it. Tell the user what command they need to run themselves and why.
3. **Is it conflict resolution explicitly commanded by the user in this conversation (Category 3)?** → Perform only the scoped steps listed there, then stop and report.
4. **Did the user waive the restriction in this message, naming the operation?** → Run exactly what was named, never a deletion, then report what ran.
5. **Is an autonomous run active (explicitly started by the user under DEVELOPMENT-WORKFLOW.md) and is the command in its allowed list?** → Run it, then continue the run.
6. **Is the command not covered by this document?** → Treat it as forbidden. Explain the situation to the user and ask how they want to proceed.

---

## Reporting Requirements

- When the agent declines a forbidden operation, it must say **which rule applies** and give the user the exact command to run manually.
- After a user-commanded conflict resolution, the agent must report: which files were conflicted, how each conflict was resolved (which side was kept, or how they were combined), and what the user still needs to do to complete the operation.
- The agent must never claim a git operation succeeded without having observed the actual command output.

---

## Summary Table

| Operation type | Examples | Agent may do it? |
| --- | --- | --- |
| Inspect / read | `status`, `log`, `diff`, `show`, `blame` | Yes, always |
| Staging / committing | `add`, `commit` | Never, user only (except `git add` on conflicted files in Category 3) |
| Remote interaction | `push`, `pull`, `fetch` | Never, user only |
| History mutation | `merge`, `rebase`, `reset`, `commit --amend`, `cherry-pick` | Never, user only |
| Working tree destruction | `clean`, `checkout --`, `restore`, `stash` | Never, user only |
| Ref/config mutation | branch/tag create-delete, `remote`, `config` writes | Never, user only |
| Conflict resolution | editing conflicted files, `git add` on those files | Only on explicit user command, scoped, then stop |
| Commit message drafting | writing/suggesting a commit message as text | Yes on request, but never with agent attribution (no AI `Co-Authored-By`, no "Generated with" footers) |
| GitHub CLI reads | `gh repo view`, `gh pr list`, `gh api` without a method | Yes, always |
| GitHub CLI writes | `gh release create`, `gh repo edit`, `gh pr merge`, `gh workflow run` | Never, user only, unless waived for that one occasion |
| Autonomous run | `init`, `add`, `commit`, local branches, private `gh repo create`, non-force `push` to the run's own repository | Only inside an explicitly started autonomous run (DEVELOPMENT-WORKFLOW.md); attribution rules still apply |
| Deleting anything | files, branches, tags, releases, repositories | Never, user only, waiver or not |
