# Worktree Workflow for CI/CD Engineers

A practical guide to using git worktrees with VS Code (1.103+) and Kiro for working on multiple Jenkins and GitLab CI branches simultaneously — without re-cloning repos or losing context when switching tasks.

---

## Why Worktrees Instead of Re-cloning

When you re-clone a repo for each task, you:
- Download all git objects again (slow, wastes disk for large repos)
- Lose any local config, IDE state, or uncommitted notes
- Have to reinstall dependencies per clone
- Cannot easily diff or copy between two branches

A git worktree gives you a **separate working directory** checked out to a different branch, but sharing the same `.git` object store as your main clone. Switching between tasks is instant — no checkout, no re-clone. Multiple worktrees can be open in separate Kiro/VS Code windows simultaneously.

```
~/.git/                    ← one set of objects, all branches
~/workspace/cicd-repo/     ← main worktree (usually main/develop)
~/workspace/cicd-hotfix/   ← linked worktree (hotfix/broken-stage)
~/workspace/cicd-feat/     ← linked worktree (feat/gitlab-template)
```

**One constraint:** The same branch can only be checked out in one worktree at a time. You can always create a new branch in a worktree.

---

## Command Reference

```bash
# Create a worktree for an existing branch
git worktree add ~/workspace/cicd-hotfix hotfix/broken-stage

# Create a worktree with a NEW branch (branching from current HEAD)
git worktree add -b feat/new-pipeline ~/workspace/cicd-feat develop

# Create a worktree and lock it (prevents accidental pruning)
git worktree add --lock ~/workspace/cicd-feat feat/thing

# List all worktrees
git worktree list
git worktree list -v    # verbose: shows locked status and reason

# Lock/unlock (to protect long-running worktrees)
git worktree lock ~/workspace/cicd-feat --reason "in active development"
git worktree unlock ~/workspace/cicd-feat

# Remove a worktree cleanly (must have no uncommitted changes unless --force)
git worktree remove ~/workspace/cicd-hotfix
git worktree remove --force ~/workspace/cicd-feat  # discard changes

# Clean up stale worktree metadata (e.g. after manually deleting a directory)
git worktree prune

# Move a worktree to a new path
git worktree move ~/workspace/cicd-feat ~/workspace/cicd-feat-v2
```

---

## Recommended Directory Layout

```
~/workspace/
├── cicd-repo/                  # Main worktree — keep on main/develop
│   ├── jenkins/
│   │   ├── shared-library/
│   │   └── pipelines/
│   └── gitlab-ci/
│
├── cicd-hotfix-<name>/         # Worktree for a hotfix
├── cicd-feat-<name>/           # Worktree for a feature
│
├── angular-monorepo/           # Reference clone — pull weekly, don't work in it
│
└── claude-to-kiro/             # Skills repo — single clone, shared by all projects
```

**Naming convention:** `cicd-<type>-<short-description>` makes `git worktree list` readable and lets you glob for cleanup (`rm -rf ~/workspace/cicd-done-*/`).

---

## VS Code and Worktrees

### Native Support (VS Code 1.103+)

VS Code added built-in git worktree support in July 2025. From the Source Control panel or Command Palette:

- `Git: Create Worktree` — creates and opens a new worktree
- `Git: Open Worktree` — opens an existing worktree in a new window
- The Repositories view shows all linked worktrees for the current repo

Each worktree opens as a standard VS Code window. It gets its own:
- Terminal sessions
- `.vscode/tasks.json` (from that worktree's directory)
- `.vscode/settings.json` (folder-level settings override workspace settings)
- Editor state (open tabs, breakpoints)

### Multi-Root Workspace for Side-by-Side Work

When you need to compare two branches or work on the monorepo alongside your CI/CD repo, use a `.code-workspace` file:

```jsonc
// ~/workspace/cicd-dev.code-workspace
{
  "folders": [
    {
      "name": "CI/CD — main",
      "path": "./cicd-repo"
    },
    {
      "name": "CI/CD — feat/pipeline-v2",
      "path": "./cicd-feat-pipeline-v2"
    },
    {
      "name": "Angular Monorepo (reference)",
      "path": "./angular-monorepo"
    }
  ],
  "settings": {
    "git.autofetch": true,
    "editor.formatOnSave": true
  },
  "tasks": {
    "version": "2.0.0",
    "tasks": [
      {
        "label": "Jenkins: Lint Jenkinsfile (active file)",
        "type": "shell",
        "command": "java -jar ${env:JENKINS_CLI_JAR} -s ${env:JENKINS_URL} declarative-linter < ${file}",
        "group": "test"
      }
    ]
  }
}
```

**Important limitation:** GitHub Copilot's `@workspace` searches only the **first root folder**. Use `#codebase` instead — it searches all roots and can be invoked multiple times within a session.

**Note:** `.github/copilot-instructions.md` is not supported in multi-root workspaces — place a `copilot-instructions.md` in each root's `.github/` folder instead, or use folder-level `.vscode/settings.json` to configure Copilot per root.

### GitLens for Worktree Visibility

GitLens (free tier) adds a Worktrees view to the Source Control sidebar. From it you can:
- See all worktrees and their current branch/dirty state
- Open a worktree in a new window with one click
- Create and delete worktrees without leaving VS Code

Install via Extensions: `eamodio.gitlens`.

### Per-Worktree VS Code Configuration

Each worktree can have its own `.vscode/` directory. Useful for CI/CD work:

```jsonc
// cicd-repo/.vscode/settings.json
{
  "files.associations": {
    "Jenkinsfile": "groovy",
    "Jenkinsfile.*": "groovy"
  },
  "editor.rulers": [120],
  "groovy.classpath": ["${workspaceFolder}/jenkins/shared-library"]
}
```

Place the same `.vscode/` in your main clone and it will carry into all worktrees that don't override it (worktrees share the `.vscode/` from their own directory, not the main repo's — so either symlink or copy it when creating a new worktree).

---

## Kiro and Worktrees

### Each Worktree Is an Independent Kiro Project

Open each worktree in a separate Kiro window. Kiro treats it as a distinct project with its own:
- Steering documents (`.kiro/steering/`)
- Hooks (`.kiro/hooks/`)
- Specs (`.kiro/specs/`)
- Session memory (`.kiro/memory/session-log.md`)

This means you can have the `feat/pipeline-v2` worktree open with a Kiro spec tracking the feature, while the `hotfix/broken-stage` worktree is open in another Kiro window with separate context.

### Multi-Root Workspaces in Kiro

Like VS Code, Kiro supports multi-root workspaces (`File > Add Folder to Workspace`). Kiro's `#codebase` searches across all roots simultaneously — no equivalent to Copilot's single-root limitation.

Each root gets its own `.kiro/` directory. Steering docs from all roots are merged by Kiro's agent, with per-root `inclusion: fileMatch` docs activating only when files from that root are being worked on.

Use multi-root in Kiro when you need the AI to understand both your CI/CD pipelines and the Angular monorepo structure they build — for example, when checking whether a pipeline change is compatible with the project's actual `angular.json` build configuration.

### Kiro Specs for Parallel Feature Work

Kiro's **Specs** system is the agent-native parallel work unit — analogous to worktrees but for AI tasks. Each spec defines requirements, a design, and a task list. The Kiro Autonomous Agent executes tasks concurrently (up to 10 in parallel) based on their dependency graph.

For CI/CD work, a spec might look like:

```markdown
<!-- .kiro/specs/pipeline-v2/requirements.md -->
# Pipeline v2 Requirements

## Goal
Migrate all Angular module pipelines from scripted to declarative syntax.

## User Stories
- As a developer, I can see build stage names in Jenkins UI (not just "sh step N")
- As a DevOps engineer, I can add new stages to a template without editing each Jenkinsfile

## Acceptance Criteria
- All Jenkinsfiles use `pipeline { }` syntax
- Shared timeout and notification config moved to shared library defaults
- All existing tests continue to pass
```

Once the spec is defined, Kiro generates a `tasks.md` with a dependency graph and can run independent tasks (converting different module Jenkinsfiles) in parallel.

### Kiro Memory Across Worktrees

The memory skill writes to `.kiro/memory/session-log.md` in the **current worktree**. To share memory across worktrees from the same repo, symlink to a shared location:

```bash
# Create shared memory location in main repo
mkdir -p ~/workspace/cicd-repo/.kiro/memory

# Symlink from each worktree
ln -s ~/workspace/cicd-repo/.kiro/memory \
      ~/workspace/cicd-feat-pipeline-v2/.kiro/memory
```

Or keep memory isolated per worktree — useful when the hotfix and feature work are genuinely separate contexts.

### Kiro Steering Shared Across Worktrees

The skills from `claude-to-kiro` don't need to be copied into every worktree. Symlink the steering and hooks directories from a single location:

```bash
# One-time setup for a new worktree
NEW_WORKTREE=~/workspace/cicd-feat-pipeline-v2
SKILLS=~/workspace/claude-to-kiro/skills

mkdir -p $NEW_WORKTREE/.kiro
ln -s $SKILLS/cicd/groovy-review/kiro/steering.md \
      $NEW_WORKTREE/.kiro/steering/40-groovy-review.md
ln -s $SKILLS/cicd/pipeline-debug/kiro/steering.md \
      $NEW_WORKTREE/.kiro/steering/41-pipeline-debug.md
ln -s $SKILLS/cicd/shared-library/kiro/steering.md \
      $NEW_WORKTREE/.kiro/steering/42-shared-library.md
ln -s $SKILLS/gstack/review/kiro/steering.md \
      $NEW_WORKTREE/.kiro/steering/30-review.md
```

Or create a shell function that does this automatically:

```bash
# Add to ~/.bashrc or ~/.zshrc
new-cicd-worktree() {
    local branch="$1"
    local name="${2:-$(echo $branch | tr '/' '-')}"
    local path=~/workspace/cicd-$name
    local skills=~/workspace/claude-to-kiro/skills

    git -C ~/workspace/cicd-repo worktree add -b "$branch" "$path" develop

    mkdir -p "$path/.kiro/steering" "$path/.kiro/hooks" "$path/.kiro/memory"
    ln -s "$skills/cicd/groovy-review/kiro/steering.md" "$path/.kiro/steering/40-groovy-review.md"
    ln -s "$skills/cicd/pipeline-debug/kiro/steering.md" "$path/.kiro/steering/41-pipeline-debug.md"
    ln -s "$skills/cicd/shared-library/kiro/steering.md" "$path/.kiro/steering/42-shared-library.md"
    ln -s "$skills/cicd/gitlab-ci/kiro/steering.md"      "$path/.kiro/steering/43-gitlab-ci.md"
    ln -s "$skills/gstack/review/kiro/steering.md"       "$path/.kiro/steering/30-review.md"
    ln -s "$skills/gstack/security/kiro/steering.md"     "$path/.kiro/steering/31-security.md"
    ln -s "$skills/memory/kiro/steering.md"              "$path/.kiro/steering/20-memory.md"

    # Hooks (copy, not symlink — hooks may need path adjustments)
    cp "$skills/cicd/groovy-review/kiro/hook.yaml" "$path/.kiro/hooks/groovy-review.yaml"
    cp "$skills/cicd/gitlab-ci/kiro/hook.yaml"     "$path/.kiro/hooks/gitlab-ci.yaml"
    cp "$skills/memory/kiro/hook.yaml"             "$path/.kiro/hooks/memory.yaml"

    echo "Worktree created: $path"
    echo "Open in Kiro:  kiro $path"
    echo "Open in VS Code: code $path"
}

# Usage:
# new-cicd-worktree feat/pipeline-v2
# new-cicd-worktree hotfix/broken-stage hotfix-broken-stage
```

---

## Kiro Autonomous Agent with Worktrees

Kiro's Autonomous Agent can run up to 10 tasks in parallel. Combined with worktrees, the recommended pattern for a large migration (e.g., converting 20 Jenkinsfiles from scripted to declarative) is:

1. **Define the spec** in one worktree with requirements and the list of files to convert
2. **Let Kiro generate tasks** — it creates one task per Jenkinsfile with dependency analysis
3. **Kiro executes tasks in parallel** — independent files are processed concurrently
4. **Review the diff per worktree** — each task's changes are staged but not committed
5. **Commit per task** — after reviewing, commit each set of changes: `git add . && git commit -m "refactor: convert module-X Jenkinsfile to declarative"`

The agent handles the repetitive mechanical work; you review and control the commit history.

---

## Day-to-Day Workflow Example

```bash
# Check what you have in flight
git -C ~/workspace/cicd-repo worktree list

# New hotfix arrives — create worktree and open immediately
new-cicd-worktree hotfix/artifact-cache-failure
kiro ~/workspace/cicd-hotfix-artifact-cache-failure &

# Continue feature work in existing window
# (already open in Kiro — Kiro session memory has context from last session)

# Done with hotfix — clean up
git -C ~/workspace/cicd-repo worktree remove ~/workspace/cicd-hotfix-artifact-cache-failure

# End of day — worktrees with in-progress work stay in place
# Next day: open Kiro in the feature worktree
# → Kiro reads .kiro/memory/session-log.md → instant context recovery
kiro ~/workspace/cicd-feat-pipeline-v2
```

---

## Common Mistakes to Avoid

| Mistake | Consequence | Fix |
|---------|------------|-----|
| Manually deleting a worktree directory | Git metadata becomes stale | Run `git worktree prune` after manual deletion |
| Checking out the same branch in two worktrees | Git refuses with "already checked out" | Create a new branch in the second worktree: `git -C <path> checkout -b my-copy` |
| Not locking a worktree during a long task | Automated `prune` may remove it | `git worktree lock <path> --reason "in use"` |
| Copying `.vscode/tasks.json` with hardcoded paths | Tasks break in new worktree location | Use `${workspaceFolder}` and `${file}` variables |
| Using `@workspace` in Copilot with multi-root | Only searches first root | Use `#codebase` instead |
| Kiro steering docs not found in new worktree | Skills not active | Use the `new-cicd-worktree` function to set up symlinks |
