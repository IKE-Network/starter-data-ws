# Workspace Goals Reference

Complete reference for `ws:` goals. Quick overview: [GOALS.md](GOALS.md).

## Convention: -draft / -publish

Most mutating goals come in pairs:
- **-draft** (default) — preview mode, no changes made
- **-publish** — executes the operation

Example: `mvn ws:feature-start-draft -Dfeature=X` previews,
`mvn ws:feature-start-publish -Dfeature=X` executes.

---

## Feature Branching

### Start: `ws:feature-start-draft` / `ws:feature-start-publish`

Create a feature branch, qualify versions (e.g., `1.0.0-SNAPSHOT` becomes
`1.0.0-CssUtils-SNAPSHOT`), cascade through BOMs and properties.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `feature` | prompted | Feature name (branch: `feature/<name>`) |
| `targetBranch` | `main` | Source branch |
| `skipVersion` | `false` | Skip version qualification |

Fails if any subproject is on a different feature branch.
Branches stay local (no auto-push).

### Track: `ws:feature-track-draft` / `ws:feature-track-publish`

Adopt a branch that **already exists** — pushed from another clone
or by another developer — in exactly the components you name.
Use this instead of `ws:switch`, which is workspace-wide and skips
any component whose target branch is not already present locally
(so a remote-only branch is skipped everywhere).

| Parameter | Default | Description |
|-----------|---------|-------------|
| `feature` | prompted | Feature name (branch: `feature/<name>`) |
| `affected` | all | Comma-separated components to move; unknown name fails |
| `remote` | `origin` | Remote to look for the branch on, and track |

Creates a tracking branch from `<remote>/feature/<name>` when the
branch is not local yet, then writes all three coherence axes —
checkout, `.ike/vcs-state`, and `workspace.yaml`'s `branch:`
fields (#904). Requires a clean tree: unlike `ws:switch` it never
auto-stashes, so adopting someone else's branch cannot bury your
work in a stash ref.

```bash
mvn ws:feature-track-publish -Dfeature=grpc_plugin \
    -Daffected=komet,tinkar-service
```

### Review: `ws:feature-pr-draft` / `ws:feature-pr-publish`

Open one pull request per component on the feature branch,
cross-linked so a reviewer who lands on any of them can reach all
the others. GitHub has no multi-repository pull request, and a
reviewer who misses a sibling approves an incomplete picture.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `feature` | prompted | Feature name |
| `affected` | all on branch | Restrict to these components |
| `base` | `main` | Branch the PRs target |
| `title` | feature name | PR title |
| `body` | — | Text prepended to every PR body |
| `reviewer` | — | Comma-separated GitHub logins to request review from |

Pushes the branch first where the remote does not have it, reuses
an already-open PR for the same head rather than duplicating (so
it is safe to re-run), then rewrites each body with the sibling
links. Requires the `gh` CLI, authenticated — the same dependency
`ws:release-publish` has.

This sits **beside** `ws:feature-finish-*`, which is unchanged.
After the PRs merge on GitHub, bring the target branch down with
`ws:refresh-main`.

### Finish: Three Strategies

**`ws:feature-finish-squash-publish`** (recommended) — single commit on target.
**`ws:feature-finish-merge-publish`** — no-ff merge, preserves history.

All strategies:
- Auto-generate commit message from per-subproject commit history
- Fail-fast if any subproject has uncommitted changes
- Strip branch-qualified versions back to base SNAPSHOT
- Accept optional `-Dmessage="summary"` prepended to auto-generated message

| Parameter | Default | Description |
|-----------|---------|-------------|
| `feature` | prompted | Feature name |
| `targetBranch` | `main` | Merge target |
| `keepBranch` | varies | Keep branch after merge |
| `message` | auto | Optional human summary |

### Abandon: `ws:feature-abandon-draft`

Delete a feature branch without merging.

### Sibling clone: `ws:feature-start-sibling-draft` / `-publish`

Clone the whole workspace into a sibling directory
(`<workspace>-<feature>`) on `feature/<name>` from inception,
instead of switching the primary in place. Each component is a
self-contained clone (`--reference --dissociate` against the
primary, so large histories are cheap). The primary stays on
its branch; the sibling is disposable (`rm -rf`) after merge.
Isolates concurrent work — same-machine or across Syncthing
machines — so two streams never stage each other's edits.
Run `-draft` first for a plan + preflight (no clone), then
`-publish` to create the sibling.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `feature` | prompted | Feature name (branch: `feature/<name>`) |
| `skipVersion` | `false` | Skip version qualification |
| `from` | current branch | Base branch to cut the sibling from; required to override when the primary is off the manifest base |

Branches and clones stay local (no auto-push).

---

## Workspace Lifecycle

| Goal | Description |
|------|-------------|
| `ws:scaffold-init` | Bootstrap a new workspace, or clone declared subprojects when workspace.yaml already exists. Safe to re-run. |
| `ws:scaffold-draft` | Check manifest, BOM cascade, VCS state (folds verify per #393) |
| `ws:verify-convergence` | Transitive dependency convergence (slow) |
| `ws:overview` | Dashboard: manifest, graph, status, cascade |
| `ws:scaffold-publish` | Apply workspace-level reconciliation (denormalized YAML field sync, scaffold conventions, etc.) |
| `ws:pull` | Git pull --rebase across components |

---

## Release & Checkpoint

| Goal | Description |
|------|-------------|
| `ws:release-draft` / `-publish` | Release release-pending components in topo order |
| `ws:checkpoint-draft` / `-publish` | Tag all subprojects, record SHAs |
| `ws:post-release` | Bump to next SNAPSHOT |
| `ws:align-draft` / `-publish` | Align inter-subproject versions |
| `ws:release-notes` | Generate notes from GitHub milestone |

---

## VCS Bridge (Syncthing)

Run on a working set of 1..N — the whole workspace, or a single
repo with no `workspace.yaml`. See `ws:help` for the per-goal
single-repo vs. workspace breakdown.

| Goal | Description |
|------|-------------|
| `ws:commit-publish` | Commit across the working set (`-Dpush=true -Dmessage="..."`) |
| `ws:push` | Push the working set (warns about uncommitted changes) |
| `ws:sync` | Pull then push across the working set |
| `ws:cleanup-draft` / `-publish` | List/delete finished (merged + squash-merged) branches |

---

## Preflight Validation

Workspace goals validate that all subproject working trees are clean
before starting. If any subproject has uncommitted changes, the goal
fails immediately with a list of affected repos and files — no partial
modifications occur.

**Goals with hard preflight (publish mode):**
`release`, `align`, `scaffold`, `checkpoint`, `pull`, `switch`,
`feature-start`, `feature-finish-*`, `feature-abandon`, `update-feature`

**Draft goals:** warn about uncommitted changes that would block the
corresponding `-publish` goal, but still run the preview.

**`ws:commit-publish`:** skips VCS bridge catch-up when there are pending
changes to commit, preventing branch-switch conflicts.

**`ws:push`:** warns about uncommitted changes after pushing, and
automatically sets upstream tracking for new branches.

**`ws:push -Dfeature=<name>`:** pushes only members currently on
`feature/<name>`; everything else is reported as "not on feature
branch" rather than silently omitted. Without the filter every
member is pushed **including the workspace root** — which fails
outright for a contributor with read-only access to the
aggregator repository, taking the whole goal down with it.

## Troubleshooting

**"Cannot X — uncommitted changes in:"** — Run `mvn ws:commit-publish -Dmessage="..."` to commit all pending changes, then retry.

**Maven discovers `.teamcity/pom.xml`** — Add `-pl !.teamcity` to `.mvn/maven.config`.

**Feature finish: "uncommitted changes"** — Run `mvn ws:commit-publish -Dmessage="..."` first.

**Feature start: "already on feature branch"** — Finish/abandon the current feature first.

**Plugin version mismatch** — After upgrading `ike-parent`, run `mvn ws:scaffold-init`.

**Stale clones on CI** — `ws:scaffold-init` now fetches and rebases existing clones. Delete subproject directories manually only if rebase conflicts occur.

---
*Generated by `ws:scaffold-init`. Regenerated when workspace plugin version changes.*
