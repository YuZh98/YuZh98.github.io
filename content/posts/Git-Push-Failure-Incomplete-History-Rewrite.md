+++
title = 'Git Push Failure: A Story of Incomplete History Rewrite'
date = 2025-12-20T19:11:00-05:00
draft = false
description = 'A hard lesson I learned with Git.'
+++

<!--more-->

## The Story

I made a large Git commit that accidentally included some big files, which caused my push to fail. To fix this, I used `git filter-repo` to remove the history of certain large sampling files. Just to be safe, I also deleted some unused local large sampling files, manually removed the cached large files that had been staged in the commit, and added the directories that might contain large files to `.gitignore`. After all that cleanup, I thought everything was ready, so I committed and pushed again.

After 15 hours, VS Code was still showing a running blue line segment as if it were still trying to push. At that point, the situation looked paradoxical. My local repository differed from the remote GitHub repository by only two commits. The first commit added a large amount of content (189 files changed and nearly 23,000 lines inserted), and this was the commit that originally caused trouble. The second commit removed much of that content, deleting over 11,000 lines and leaving the working tree clean. Running `git status` showed no pending changes, and inspecting the current tree confirmed that no large files remained.

Initially, it even appeared that no push was happening at all. Running `git remote -v` on a separate terminal window revealed that no remote repository was configured, meaning that the apparent "15-hour push" was just a misleading VS Code UI state. After re-adding the GitHub remote, a real push attempt finally began.

That's when the real problem surfaced.

Although the *current* commit history looked clean, Git attempted to upload a massive 12.66-gigabyte pack and promptly failed. This revealed a crucial detail that had been easy to miss: although I had used `git filter-repo`, I had **not removed all large files from the repository history**. In particular, large MCMC output files living under `demo_results/`—some exceeding 12 GB individually—were still present as blobs in an earlier commit. They had been deleted later, but they remained reachable in the commit graph.

This explained everything. Git does not push diffs or final snapshots; it pushes **all objects reachable from the commits being sent**. A file that is added in one commit and deleted in the next still exists in history and must be transferred. Commands like `git diff` only show net effects and therefore completely hide this class of problem.

At this point, two valid solutions existed. One option was to re-run `git filter-repo` correctly, this time explicitly removing *all* large artifacts (especially the `demo_results/` directories) from history. Had this been done, the push would have succeeded: rewriting history **would have worked**, provided the rewrite was complete.

Instead of immediately rewriting history again, I paused to verify what exactly was happening. We carefully inspected the repository to locate the largest blobs and confirmed their origin. I verified that the problematic commits had never been pushed to GitHub, meaning no collaborators or published history would be affected. With that assurance, we chose a safer and simpler alternative.

Rather than rewriting history again, I dropped the problematic commits entirely. The main branch was reset to the remote base, and only the intended source code, notebooks, and documentation were selectively restored from a backup branch. All generated artifacts including sampling outputs, cache files, and experimental results were explicitly excluded, and `.gitignore` was strengthened to prevent future accidents.

After rebuilding the commit cleanly and verifying that no large blobs remained in the commit range, the repository was pushed again. This time, the push completed almost instantly.

In the end, the failure wasn't caused by Git behaving unexpectedly, nor by history rewriting being ineffective. The real issue was **an incomplete history rewrite** combined with Git's object-based push semantics.

---

## Technical Report: Diagnosing and Resolving a Git Push Failure Caused by Incomplete History Rewrite

### 1. Initial Cleanup Attempt (Incomplete)

After the first push failed due to large files, history rewrite was performed:

```bash
# Remove specific paths from history
git filter-repo --path path/to/large/output/ --invert-paths

# Delete local large files manually
rm -rf path/to/local/large/files

# Update .gitignore (changes later lost)
echo "large_output_dir/" >> .gitignore

# Stage and commit
git add -u
git add .gitignore
git commit -m "Remove large files and update .gitignore"
```

**Problem:** This removed *some* large files but not all. Large sampling outputs in other directories remained in history.

### 2. Misleading Initial Diagnosis

After cleanup, surface-level checks suggested the repository was clean:

```bash
git status                     # Clean working tree
git diff origin/main..main     # Returned empty (no visible diff)
git log origin/main..main      # Returned empty (appeared up-to-date)
```

However, these commands only inspect the *current snapshot*, not the full object graph.

### 3. Remote Configuration Issue

`git filter-repo` removes the remote configuration by design:

```bash
git remote -v                  # Returned nothing
```

The "15-hour push" in VS Code was actually a misleading UI state—no push was happening at all. After discovering this:

```bash
# Re-add the remote
git remote add origin https://github.com/user/repo.git

# Fetch to establish tracking
git fetch origin
```

### 4. The Real Problem Surfaces

After re-adding the remote and attempting to push:

```bash
git push -u origin main
```

Output:
```
Enumerating objects: 227, done.
Counting objects: 100% (227/227), done.
Delta compression using up to 8 threads
Compressing objects: 100% (141/141), done.
error: RPC failed; HTTP 400 curl 22 The requested URL returned error: 400
send-pack: unexpected disconnect while reading sideband packet
Writing objects: 100% (216/216), 12.66 GiB | 359.26 MiB/s, done.
Total 216 (delta 81), reused 200 (delta 70), pack-reused 0
fatal: the remote end hung up unexpectedly
Everything up-to-date
```

Git attempted to upload 12.66 GB and failed. The confusing "Everything up-to-date" message appeared because the push failed mid-transfer.

**Key insight:** Git pushes all objects reachable from commits, not diffs. A file added in commit A and deleted in commit B still exists in history and must be transferred.

### 5. Diagnosing the Incomplete Rewrite

Inspection of large blobs in the commit range being pushed:

```bash
# Find largest blobs in commits to be pushed
git rev-list --objects origin/main..main \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '$1=="blob"{print $3 "\t" $4}' \
  | sort -nr \
  | head -n 30
```

**Result (actual output):**
```
13230021344     path/to/large_samples_1.npz
1230020746      path/to/large_samples_2.npz
901801300       path/to/large_samples_3.npz
181800826       path/to/large_samples_4.npz
150301620       path/to/large_samples_5.npz
96201420        path/to/large_samples_6.npz
83085391        path/to/large_samples_7.npz
...
```

A single file exceeded 13 GB (13,230,021,344 bytes ≈ 12.3 GiB). Multiple other files ranged from hundreds of megabytes to several gigabytes. All were `.npz` (NumPy compressed array) files containing MCMC sampling outputs.

### 6. Root Cause

The `filter-repo` command only removed one specific directory. Large sampling output files in other directories were never filtered, remaining reachable in the commit graph.

**Alternative solution (not taken):** A second, complete history rewrite:

```bash
# This would have worked if all large files were removed
git filter-repo --strip-blobs-bigger-than 100M

# Or remove all output directories
git filter-repo --path 'output_dir_1/' --invert-paths \
                --path 'output_dir_2/' --invert-paths \
                --path 'sampling_results/' --invert-paths
```

### 7. Chosen Solution: Drop Problematic Commits Entirely

Since the problematic commits were never pushed to GitHub, a simpler and safer approach was taken:

```bash
# Step 1: Create backup branch to preserve work
git switch -c backup/with-bad-commits

# Step 2: Switch back and reset main to remote state
git switch main
git reset --hard origin/main

# Step 3: Selectively restore desired files from backup
# Use multiple checkout commands to restore specific files/directories
git checkout backup/with-bad-commits -- path/to/source/code/
git checkout backup/with-bad-commits -- notebooks/
git checkout backup/with-bad-commits -- documentation/
git checkout backup/with-bad-commits -- .gitignore

# Step 4: Remove unwanted files that were restored
git rm -r --cached path/to/__pycache__
git rm --cached large_binary_file.zip
git rm --cached generated_image.png

# Step 5: Verify staging area and update .gitignore
git status

# Add final .gitignore updates
git add .gitignore

# Step 6: Verify no large files remain
git add -A  # Stage all remaining changes
```

**Note:** The actual workflow involved iterative `git checkout backup/with-bad-commits -- <path>` commands to selectively restore files, followed by `git rm --cached` commands to exclude unwanted files like `__pycache__/` directories, binary assets, and generated outputs.

### 8. Verification Before Push

Critical verification to ensure no large blobs remain:

```bash
# Check for large blobs in new commit range
git rev-list --objects origin/main..main \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '$1=="blob"{print $3 "\t" $4}' \
  | sort -nr \
  | head -n 10
```

**Result:** Command returned empty output, confirming no blobs in the commit range (only modifications to existing files).

```bash
# Final status check
git status
```

Showed only source code, notebooks, documentation files, and updated `.gitignore` staged for commit.

```bash
# Commit the clean version
git commit -m "Add new features and update .gitignore

- Add source code and notebooks
- Update documentation
- Strengthen .gitignore to prevent tracking generated artifacts"
```

### 9. Successful Push

```bash
git push -u origin main
```

**Output:**
```
Enumerating objects: 40, done.
Counting objects: 100% (40/40), done.
Delta compression using up to 8 threads
Compressing objects: 100% (30/30), done.
Writing objects: 100% (31/31), 110.92 KiB | 110.92 MiB/s, done.
Total 31 (delta 6), reused 15 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (6/6), completed with 5 local objects.
To https://github.com/user/repo.git
   c62d70f..ad6399f  main -> main
branch 'main' set up to track 'origin/main'
```

Push completed in seconds. Total transfer was only ~111 KB instead of 12.66 GB.

### Key Lessons

1. **Git pushes objects, not diffs.** Files deleted in later commits still exist in history if reachable from any commit being pushed. The "Everything up-to-date" message can appear even when a push fails mid-transfer.

2. **Surface checks are insufficient.** Commands like `git status`, `git diff`, and even `git log origin/main..main` can return empty results while large blobs remain in the object graph. Always verify with:
   ```bash
   git rev-list --objects origin/main..main | \
     git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
     awk '$1=="blob"{print $3 "\t" $4}' | sort -nr | head -n 30
   ```

3. **History rewriting must be complete.** An incomplete `filter-repo` operation leaves problematic blobs in history. Size-based filtering is often safer than path-based filtering:
   ```bash
   git filter-repo --strip-blobs-bigger-than 100M
   ```

4. **`git filter-repo` removes remote configuration.** After running `filter-repo`, remotes must be re-added:
   ```bash
   git remote add origin <url>
   git fetch origin
   ```

5. **Unpublished commits can simply be dropped.** When problematic commits haven't been pushed, resetting to `origin/main` and selectively restoring files is safer and simpler than multiple history rewrites.

6. **Iterative restoration workflow is practical.** Using multiple `git checkout <branch> -- <path>` commands followed by selective `git rm --cached` commands provides fine-grained control over what gets committed.

7. **Verify before pushing.** Always inspect reachable objects in the push range, not just the working tree. An empty result from the blob inspection command confirms safety.

### Quick Reference Commands

```bash
# Check remote configuration
git remote -v

# Re-add remote after filter-repo
git remote add origin <url>
git fetch origin

# Find merge base (common ancestor)
git merge-base main origin/main

# Find large files in commits being pushed
git rev-list --objects origin/main..main \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '$1=="blob"{print $3 "\t" $4}' \
  | sort -nr \
  | head -n 30

# Create backup and reset
git switch -c backup/with-bad-commits
git switch main
git reset --hard origin/main

# Selectively restore files
git checkout backup/with-bad-commits -- path/to/restore/

# Remove unwanted files from staging
git rm --cached unwanted_file
git rm -r --cached unwanted_directory/

# Verify before committing
git status
git diff --cached --stat

# Remove large files from entire history (alternative approach)
git filter-repo --strip-blobs-bigger-than 100M
```
