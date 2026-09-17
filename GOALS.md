# Workspace Goals

All goals are available in IntelliJ's Maven tool window
under **Plugins > ws** and **Plugins > ike**.

## Workspace Management

| Goal | Description |
|------|-------------|
| `ws:scaffold-init` | Bootstrap a new workspace (no manifest yet) or clone declared subprojects (manifest present). Idempotent. |
| `ws:add` | Add a subproject repo (prompts for URL) |
| `ws:scaffold-publish` | Reconcile workspace state (versions, groupIds, scaffold conventions) — subsumes the retired scaffold-upgrade goals |
| `ws:graph` | Print dependency graph (text or DOT format) |
| `ws:stignore` | Generate Syncthing ignore rules |
| `ws:remove` | Remove a subproject (prompts for name) |
| `ws:help` | List all ws: goals with descriptions |

## Verification

| Goal | Description |
|------|-------------|
| `ws:scaffold-draft` | Check manifest, parents, BOM cascade, VCS state (folds verify per #393) |
| `ws:verify-convergence` | Transitive dependency convergence (slow) |
| `ws:overview` | Workspace overview (manifest, graph, status, cascade) |
| `ws:check-branch` | Warn when a subproject branch deviates from workspace.yaml |

## Version Alignment

| Goal | Description |
|------|-------------|
| `ws:align-draft` | Preview inter-subproject POM version alignment (AlignmentReconciler) |
| `ws:align-publish` | Apply POM version alignment |
| `ws:reconcile-branches-draft` / `-publish` | Reconcile branch fields against on-disk state |
| `ws:scaffold-draft -DupdateParent=true` | Preview parent-POM version cascade (along with other reconciliation) |
| `ws:scaffold-publish -DparentVersion=<v>` | Pin parent to specific version and cascade |

## Branch Coordination

| Goal | Description |
|------|-------------|
| `ws:switch-draft` | Preview switching subprojects to a coordinated branch |
| `ws:switch-publish` | Switch subprojects to a coordinated branch |
| `ws:update-feature-draft` | Preview rebasing a feature branch onto main |
| `ws:update-feature-publish` | Rebase a feature branch onto main |

## Feature Branching

| Goal | Description |
|------|-------------|
| `ws:feature-start-draft` | Preview feature branch |
| `ws:feature-start-publish` | Create feature branch across components |
| `ws:feature-track-draft` | Preview adopting an existing branch in selected components |
| `ws:feature-track-publish` | Adopt an existing branch (`-Daffected=a,b`), creating it from the remote |
| `ws:feature-pr-draft` | Preview the review pull requests |
| `ws:feature-pr-publish` | Open cross-linked review PRs, one per component |
| `ws:feature-finish-merge-draft` | Preview no-ff merge |
| `ws:feature-finish-merge-publish` | No-ff merge (preserves history) |
| `ws:feature-finish-squash-draft` | Preview squash merge |
| `ws:feature-finish-squash-publish` | Squash merge (single commit) |
| `ws:feature-abandon-draft` | Preview abandoning a feature branch |
| `ws:feature-abandon-publish` | Delete feature branch across components |
| `ws:feature-start-sibling-draft` | Preview a sibling-clone feature start (no clone) |
| `ws:feature-start-sibling-publish` | Start a feature in a sibling clone beside the primary (isolated, Syncthing-safe) |

## Release & Checkpoint

| Goal | Description |
|------|-------------|
| `ws:release-draft` | Preview what would be released |
| `ws:release-publish` | Execute workspace release |
| `ws:release-status` | Diagnose state of any in-flight workspace release |
| `ws:checkpoint-draft` | Preview checkpoint (tag all subprojects) |
| `ws:checkpoint-publish` | Execute checkpoint |
| `ws:post-release` | Bump to next development version |
| `ws:release-notes` | Generate release notes from GitHub milestone |

## VCS Bridge (Syncthing multi-machine)

These run on a working set of 1..N — every subproject in a
workspace, or a single repo with no `workspace.yaml`. Run
`ws:help` for the per-goal single-repo vs. workspace breakdown.

| Goal | Description |
|------|-------------|
| `ws:sync` | Pull then push across the working set (the daily sync op) |
| `ws:commit-draft` | Preview what would be committed across the working set (read-only) |
| `ws:commit-publish` | Commit across the working set (stages all by default; `-DstagedOnly` to opt out) |
| `ws:pull` | Git pull --rebase across the working set |
| `ws:push` | Push the working set (warns about uncommitted changes) |
| `ws:report` | List the ws:* goal reports for the working set |

## Branch Cleanup

| Goal | Description |
|------|-------------|
| `ws:cleanup-draft` | List finished (merged + squash-merged) feature branches |
| `ws:cleanup-publish` | Delete finished feature branches, local and remote |

## Build Goals (ike:)

| Goal | Description |
|------|-------------|
| `ike:release-draft` | Preview single-repo release |
| `ike:release-publish` | Execute single-repo release |
| `ike:generate-bom` | Generate BOM with resolved versions |
| `ike:site-draft` | Preview site deployment + org-site registration |
| `ike:site-publish` | Deploy project site and register it on the org site |
| `ike:help` | List all ike: goals with descriptions |

---
*Generated by `ws:scaffold-init`. See `ws:help` and `ike:help` for full details.*
