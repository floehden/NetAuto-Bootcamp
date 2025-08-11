# Day 4: Merging Branches

**Goal:** Learn how to merge branches.

1. **Merge `feature-branch` into `main`:**
   - Make sure you are on the `main` branch:
     ```bash
     git checkout main
     ```
   - Merge the `feature-branch` into `main`:
     ```bash
     git merge feature-branch
     ```

2. **Resolve Merge Conflicts (if any):**
   - If there are any conflicts, resolve them manually by editing the conflicting files.
   - Add and commit the resolved files:
     ```bash
     git add <file>
     git commit -m "Resolved merge conflicts"
     ```
