# Syncing with Upstream Fuwari

This repository is a fork of [saicaca/fuwari](https://github.com/saicaca/fuwari). This document explains how to keep your fork synchronized with the upstream repository.

## Configuration

The repository is configured with two remotes:
- `origin`: Your fork (rosmontisu/Blog)
- `upstream`: The original repository (saicaca/fuwari)

## Important Note About Unrelated Histories

This Blog repository has a separate git history from the upstream fuwari repository. When syncing for the first time or when the histories are unrelated, you must use the `--allow-unrelated-histories` flag.

## How to Sync with Upstream

### Option 1: Merge Upstream Changes (Recommended)

This method preserves your commit history:

```bash
# 1. Fetch the latest changes from upstream
git fetch upstream

# 2. Switch to your main branch
git checkout main

# 3. Merge upstream changes into your main branch
# NOTE: Use --allow-unrelated-histories if this is the first time syncing
# or if you get "refusing to merge unrelated histories" error
git merge upstream/main --allow-unrelated-histories

# 4. Resolve any merge conflicts if they occur
# (Edit conflicting files, then git add them)
# There will likely be many conflicts on the first sync

# 5. Complete the merge
git merge --continue

# 6. Push the changes to your fork
git push origin main
```

### Option 2: Rebase onto Upstream (Advanced - Not Recommended for Unrelated Histories)

**Warning:** This method is not recommended for this repository since the histories are unrelated. Use Option 1 (merge) instead.

This method creates a cleaner history but rewrites commits and doesn't work well with unrelated histories:

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

The recommended workflow for syncing with upstream safely:

1. Create a new branch from your main:
   ```bash
   git checkout main
   git checkout -b sync-upstream
   ```

2. Merge with upstream (use --allow-unrelated-histories if needed):
   ```bash
   git merge upstream/main --allow-unrelated-histories
   ```

3. Resolve all conflicts carefully:
   - Review each conflict
   - Keep your custom configurations (src/config.ts, etc.)
   - Accept new features and bug fixes from upstream
   - Test the merged code locally

4. Push the branch and create a PR on GitHub:
   ```bash
   git push origin sync-upstream
   ```

5. Review and merge the PR through GitHub's interface

## Recommended Sync Strategy for This Repository

Given that this repository has unrelated histories with upstream fuwari, here's the recommended approach:

### For the First Major Sync:

1. **Review upstream changes first:** Check what's new in upstream/main
2. **Cherry-pick specific features:** Instead of merging everything, cherry-pick specific commits you want
3. **Manual updates:** For major features, consider manually implementing them to avoid extensive conflicts

### For Ongoing Syncs:

Once you've done an initial sync with `--allow-unrelated-histories`, future syncs will be easier:

1. Regularly fetch upstream: `git fetch upstream`
2. Create sync branches: `git checkout -b sync-upstream-YYYYMMDD`
3. Merge incrementally: Sync more frequently to reduce conflicts
4. Test thoroughly: Always test locally before pushing

## Troubleshooting

### "refusing to merge unrelated histories"
This is expected for this repository. Use the `--allow-unrelated-histories` flag:
```bash
git merge upstream/main --allow-unrelated-histories
```

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
