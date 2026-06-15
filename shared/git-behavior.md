## Git behavior

**Confirm before large commits.** Before running `git commit`, check `git diff --staged --stat`. If the total lines changed (insertions + deletions) is ≥ 50, or ≥ 5 files are changed, show the stat summary and proposed commit message and wait for approval. Below that threshold, commit directly.
