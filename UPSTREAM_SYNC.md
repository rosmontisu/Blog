# Syncing with Upstream Fuwari

This repository is a fork of [saicaca/fuwari](https://github.com/saicaca/fuwari). This document explains how to keep your fork synchronized with the upstream repository.

## Configuration

The repository is configured with two remotes:
- `origin`: Your fork (rosmontisu/Blog)
- `upstream`: The original repository (saicaca/fuwari)

## How to Sync with Upstream

### Option 1: Merge Upstream Changes (Recommended)

This method preserves your commit history:

```bash
# 1. Fetch the latest changes from upstream
git fetch upstream

# 2. Switch to your main branch
git checkout main

# 3. Merge upstream changes into your main branch
git merge upstream/main

# 4. Resolve any merge conflicts if they occur
# (Edit conflicting files, then git add them)

# 5. Push the changes to your fork
git push origin main
```

### Option 2: Rebase onto Upstream

This method creates a cleaner history but rewrites commits:

```bash
# 1. Fetch the latest changes from upstream
git fetch upstream

# 2. Switch to your main branch
git checkout main

# 3. Rebase your changes onto upstream
git rebase upstream/main

# 4. Resolve any conflicts if they occur
# (Edit conflicting files, then git add them and git rebase --continue)

# 5. Force push the changes (be careful with this!)
git push origin main --force-with-lease
```

### Option 3: Cherry-pick Specific Commits

If you only want specific changes from upstream:

```bash
# 1. Fetch the latest changes from upstream
git fetch upstream

# 2. View upstream commits
git log upstream/main

# 3. Cherry-pick specific commits (replace COMMIT_HASH with actual hash)
git cherry-pick COMMIT_HASH

# 4. Push the changes
git push origin main
```

## Checking for Updates

To see what's new in upstream:

```bash
# Fetch upstream changes
git fetch upstream

# Compare your main branch with upstream
git log origin/main..upstream/main --oneline

# See detailed differences
git diff origin/main upstream/main
```

## Creating Pull Requests from Upstream

If you want to review upstream changes before merging:

1. Create a new branch from your main:
   ```bash
   git checkout main
   git checkout -b sync-upstream
   ```

2. Merge or rebase with upstream:
   ```bash
   git merge upstream/main
   # or
   git rebase upstream/main
   ```

3. Push the branch and create a PR on GitHub:
   ```bash
   git push origin sync-upstream
   ```

4. Review and merge the PR through GitHub's interface

## Troubleshooting

### "Your branch has diverged"
This means your main and upstream/main have different commits. You need to either merge or rebase.

### Merge Conflicts
When conflicts occur:
1. Open the conflicting files
2. Look for conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
3. Resolve the conflicts by choosing which changes to keep
4. Run `git add <resolved-files>`
5. Complete the merge with `git merge --continue` or rebase with `git rebase --continue`

### Failed to Push
If push fails, it might be because:
- You need to pull changes first: `git pull origin main`
- You're rebasing and need to force push: `git push origin main --force-with-lease`

## Notes

- The upstream remote is already configured in this repository
- Always fetch from upstream before attempting to sync
- Consider creating a backup branch before syncing: `git checkout -b backup-main`
- Test thoroughly after syncing before deploying
