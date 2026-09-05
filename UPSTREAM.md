# Upstream fork maintenance

This repository is a **true GitHub fork** of [tcarac/taskboard](https://github.com/tcarac/taskboard). Keep `fork: true` and that parent. Do not convert it to a standalone repo.

Factory-owned behavior lives on `main` as first-class commits. Upstream moves in through `chore: sync upstream` pull requests. Never force-push `main`.

## Remotes

| Remote | URL | Role |
| --- | --- | --- |
| `origin` | https://github.com/atebites-hub/taskboard.git | This fork (push / PRs) |
| `upstream` | https://github.com/tcarac/taskboard.git | Parent (fetch only) |

```bash
git remote add origin https://github.com/atebites-hub/taskboard.git   # if missing
git remote add upstream https://github.com/tcarac/taskboard.git      # if missing
git remote -v
# origin    https://github.com/atebites-hub/taskboard.git (fetch/push)
# upstream  https://github.com/tcarac/taskboard.git (fetch)
```

Do not `git push` to `upstream`.

## Last synced upstream tip

- **Upstream:** https://github.com/tcarac/taskboard
- **Parent:** [tcarac/taskboard](https://github.com/tcarac/taskboard)
- **Last synced upstream tip:** <!-- upstream-tip-begin -->`c3db6eba367edce32c9942362f707dd7f91ea689` (`c3db6eb`, `chore: Cleanup and screenshot (#1)`)<!-- upstream-tip-end -->

The weekday sync workflow rewrites only the `upstream-tip-begin/end` span when it opens a clean sync PR.

## Owners

- **Jaskarn** (atebites-hub)
- **Factory Plugins bot**

## Divergence (patches we own)

These are atebites-only. Do not drop them in an upstream merge without recording the deferral here.

| Patch / behavior | Why we keep it | Conflict risk | Source |
| --- | --- | --- | --- |
| Claude Code plugin (marketplace + skill + MCP + fail-open hooks) | Per-repo SQLite at `.taskboard/taskboard.db` instead of the OS config-dir default | Low — new files; Medium if README / plugin layout is touched upstream | `ed227b3` |
| README fork note | Points consumers at [PLUGIN.md](PLUGIN.md) and this file | Low | `ed227b3` |
| Artifact retention (3 days) + weekly cleanup | Caps Actions storage for release binaries; weekly Monday UTC job deletes leftovers older than 3 days | Low — `retention-days` on `release.yml` plus new `.github/workflows/cleanup-artifacts.yml` | this fork |

Do not bump [atebites-plugins](https://github.com/atebites-hub/atebites-plugins) `plugins/taskboard/upstream` pins from this repository.

## Deferred (intentionally not in this fork yet)

| Item | Reason |
| --- | --- |
| atebites-plugins pin bumps | Separate PR after this fork's CI is green and the smoke matrix has been attempted |

## Sync policy (Project Factory FORK-MAINTENANCE)

1. **Keep the GitHub fork relationship.** Parent must stay `tcarac/taskboard`.
2. **Never rewrite published `main`.** No force-push to `main`.
3. **Do not rebase factory commits off `main`.** Replay happens by *merging* `upstream/main` into a branch that already has factory commits.
4. **Sync through a PR titled exactly `chore: sync upstream`** into `main`. Prefer GitHub **Create a merge commit** (not squash, not rebase) so factory SHAs stay reachable and the next merge has a sane merge-base.
5. **Preserve factory files.** Keep `.claude-plugin/`, `PLUGIN.md`, `.mcp.json`, `hooks/`, `scripts/session-sync.sh`, `scripts/commit-sync.sh`, and `skills/taskboard-workflow/`.
6. **Update this file** after each successful sync: last synced tip (the `upstream-tip` markers) and any new divergence or deferral.

### Manual sync

```bash
git fetch origin
git fetch upstream
git checkout -b chore/sync-upstream-$(git rev-parse --short upstream/main) origin/main

# Skip if we already contain upstream/main:
#   git merge-base --is-ancestor upstream/main HEAD && echo already synced

git merge --no-ff upstream/main -m "chore: merge upstream $(git rev-parse --short upstream/main)"
# Resolve conflicts using the divergence table. Keep factory plugin files.
# Update the Last synced upstream tip markers in this file.

git push -u origin HEAD
# Open PR title: chore: sync upstream
# Merge with a merge commit.
```

Weekday automation: [`.github/workflows/sync-upstream.yml`](.github/workflows/sync-upstream.yml) (UTC cron `41 13 * * 1-5`, plus `workflow_dispatch`). If an open PR already has that exact title, the workflow leaves it alone.

`GITHUB_TOKEN` pull requests do not start other workflows. Set repository secret `UPSTREAM_SYNC_TOKEN` (Factory Plugins bot PAT with `contents` + `pull-requests`) so sync PRs still run CI.

### After every sync

- [ ] Plugin + hooks + skill still present (or documented deferred)
- [ ] This file’s last-synced SHA matches `upstream/main`
- [ ] Fork still `fork: true` with parent `tcarac/taskboard`
- [ ] No marketplace pin bump from this change
