### Repository Structure Overview

**1. SRC Repository**:
   - **Main (`main`)**: Continuous development branch with selected features and fixes merged from feature and bugfix branches.
   - **Feature/Bugfix Branches**: Used for development of specific features or fixes that can be merged into multiple customer branches.
   - **Customer Branches**: Aggregate features and fixes for specific sites; includes necessary source and config for live systems.
   - **Release Tags**: Mark stable/tested versions of the core product.

**2. Walmart Repository** (Fork of SRC):
   - **Customer Branches**: Similar to SRC but specific to Walmart.
   - **Group/Develop (`group/develop`)**: Acts as the main branch within this fork.

### Detailed Workflow for Walmart repository

#### **Creating and Managing Branches**

1. **Creating a Feature/Bugfix Branch:**
   ```bash
   git checkout -b feature/dops-123456 release/osr-6-2-1
   # OR
   git checkout -b bugfix/dops-123456 release/osr-6-2-1
   ```
  This command bases the new feature/bugfix branch on a stable release, the branch is only present locally.

  **Push to remote**
 ```bash
  git push origin feature/dops-123456
  # OR
  git push origin bugfix/dops-123456
  ```
  This command pushes our branch to remote. It's now available for everyone!
   
   **Handling Special Case Fixes:**
   
- If a bug is specific to a feature branch:
  ```bash
  git checkout -b bugfix/dops-123456 feature/zdp-100032460.example
  ```

2. **Once development is done check for conflicts with group/develop:**
   ```bash
   git checkout group/develop
   git merge --no-commit --no-ff feature/dops-123456
   # OR
   git merge --no-commit --no-ff bugfix/dops-123456
   ```
   **If merge conflicts are present -> step 3 else -> step 4**
   
3. **Merge conflicts:**
   ```bash
   git checkout -b merge/feature/dops-123456 group/develop
   git merge --no-commit --no-ff feature/dops-123456
   # OR
   git merge --no-commit --no-ff bugfix/dops-123456
   ```
   
4. **Creating a Merge Request in GitLab:**
   - Go to your branch in GitLab.
   - Click the `Create merge request` button.
   - Click on the `Change branches` link/button.
   - Select the branch you based your fix on as the `Target Branch`.
   - Assign the merge request to `Michael Lang (@lang9)`.
   - Click the `Create merge request` button.

### Workflow for SRC is similar

**Most used branches, on which we will base our branches for SRC**
- feature/zdp-100023559.hive
- feature/zdp-100023559.sequencing
- feature/zdp-100023559.multi-steuer

#### **Tracking and Identifying Commit Origins**

**Finding the Original Branch of a Commit:**
- **List Branches Containing a Specific Commit:**
  ```bash
  git branch --all --contains <commitHash>
  ```

- **Custom Alias to Find Merge Commit:**
  Add this to your `.gitconfig`:
  ```ini
  [alias]
  find-merge = "!sh -c 'commit=$0 && branch=${1:-HEAD} && (git rev-list $commit..$branch --ancestry-path | cat -n; git rev-list $commit..$branch --first-parent | cat -n) | sort -k2 -s | uniq -f1 -d | sort -n | tail -1 | cut -f2'"
  ```

  Use it like this:
  ```bash
  git find-merge <commitHash> <branch>
  # example
  git find-merge 66a8d15f833f2efb892b84afbcf807fee5d46a93 origin/merge/feature/walmart-zdp-100023559.hive
  # view the result at https://git.knapp.at/osr/project/walmart/-/commit/enter_commit_here
  ```

#### **Setup Local Git Config for SRC and WALMART Repositories:**
Add the following to your `.git/config`:
```ini
[remote "origin"]
    url = git@git.knapp.at:osr/project/walmart
    fetch = +refs/heads/*:refs/remotes/origin/*
[remote "src"]
    url = git@git.knapp.at:osr/src
    fetch = +refs/heads/*:refs/remotes/src/*
```

This setup allows you to fetch, merge, and work with both repositories.

#### **Deleting Branches**

- **Delete Branch Locally and Remotely:**
  ```sh
  git branch -d bugfix/dops-288966
  git push origin --delete bugfix/dops-288966
  ```

### Pull Requests

**Best Practices:**
- Regularly update your branches from their upstream source to minimize conflicts.
- Always add relevant tags and notes in your commits to help track changes through DOPS or similar systems.
- Use merge requests as a review stage before final integration to ensure quality and traceability.
