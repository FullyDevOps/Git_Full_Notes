## 🌿 Comprehensive Guide to Git Branching & Merging (Section 03)  
*Everything you need to know, explained clearly for future reference.*

---

### **I. Introduction to Branches**  
#### **What is a Branch?**  
- **Definition**: A branch is a **lightweight, movable pointer** to a specific commit. It’s *not* a copy of files—it’s a reference to a snapshot in Git’s commit history.  
- **Why Branches?**  
  - Isolate work (e.g., new features, bug fixes) without disrupting `main`.  
  - Enable parallel development.  
  - Provide a "safe space" to experiment.  
- **Under the Hood**:  
  - Git stores branches as files in `.git/refs/heads/`.  
  - The `HEAD` file points to your **current branch** (e.g., `ref: refs/heads/feature-login`).  

#### **The `main`/`master` Branch**  
- **Purpose**: The **default branch** created when initializing a repo (`git init`). Represents the "production-ready" state.  
- **Naming Note**:  
  - Historically named `master`, but modern repos use `main` (driven by inclusivity initiatives).  
  - Configured via `git config init.defaultBranch main`.  
- **Golden Rule**: *Never commit directly to `main` in professional workflows.* Always use branches.  

#### **Creating & Switching Branches**  
| Command | Action | Example |  
|---------|--------|---------|  
| `git branch <branch>` | Creates a **new branch** (doesn’t switch) | `git branch feature-auth` |  
| `git switch <branch>` | Switches to an **existing branch** | `git switch feature-auth` |  
| `git checkout -b <branch>` | **Creates + switches** to a new branch | `git checkout -b feature-auth` |  
| `git switch -c <branch>` | Modern equivalent of `checkout -b` | `git switch -c feature-auth` |  

> 💡 **Key Insight**: Switching branches updates your **working directory** to match the commit the branch points to. Uncommitted changes may cause errors!

---

### **II. Branch Management**  
#### **Listing Branches**  
- `git branch`: Lists **all local branches**.  
  - `*` marks the **current branch**.  
  - `git branch -v`: Shows **last commit message** for each branch.  
  - `git branch --merged`: Lists branches **already merged** into current branch (safe to delete).  
  - `git branch --no-merged`: Lists branches **not merged** (deleting loses work!).  

#### **Deleting Branches**  
| Command | Action | When to Use |  
|---------|--------|-------------|  
| `git branch -d <branch>` | **Safe delete** (only if merged) | After merging a feature branch |  
| `git branch -D <branch>` | **Force delete** (ignores merge status) | Abandoned branches or mistakes |  

> ⚠️ **Critical Warning**:  
> - `-d` prevents accidental deletion of unmerged work.  
> - `-D` is **dangerous**—use only when you’re 100% sure the branch is obsolete.  

#### **Renaming Branches**  
- `git branch -m <old> <new>`: Renames a **local branch**.  
  - Example: `git branch -m feature-login feat/auth`  
- For **remote branches**:  
  1. Rename locally: `git branch -m old-branch new-branch`  
  2. Delete old remote branch: `git push origin --delete old-branch`  
  3. Push new branch: `git push origin new-branch`  
  4. Reset upstream: `git push -u origin new-branch`  

---

### **III. Merging Branches**  
#### **Fast-Forward Merge**  
- **When it happens**: When the target branch (e.g., `main`) **has no new commits** since the feature branch was created.  
- **Result**: `main` pointer **moves forward** to the tip of the feature branch (linear history).  
- **Command**: `git merge feature-auth`  
- **Visualization**:  
  ```plaintext
  A --- B --- C (main)        A --- B --- C --- D (main, feature-auth)
           \       === FF ===>
            D (feature-auth)
  ```  
- **Disable FF**: `git merge --no-ff feature-auth` (creates a merge commit even if FF is possible).  

#### **3-Way Merge**  
- **When it happens**: When **both branches have new commits** (non-linear history).  
- **How it works**:  
  1. Git finds the **common ancestor** (commit `B`).  
  2. Compares changes from `B → C` (main) and `B → D` (feature).  
  3. Creates a **new merge commit** combining both.  
- **Visualization**:  
  ```plaintext
  A --- B --- C (main)
           \ 
            D --- E (feature-auth)
  
  Merge: A --- B --- C --- F (main)
                 \       /
                  D --- E
  ```  

#### **Merge Conflicts**  
- **Cause**: When **the same part of a file** is modified differently in both branches.  
- **Detection**:  
  - Git halts the merge and marks conflicted files:  
    ```plaintext
    <<<<<<< HEAD
    main branch code
    =======
    feature branch code
    >>>>>>> feature-auth
    ```  
- **Resolution Steps**:  
  1. Open conflicted files in an editor.  
  2. **Manually edit** to keep desired changes (delete `<<<<<<<`, `=======`, `>>>>>>>`).  
  3. `git add <resolved-file>` to mark as resolved.  
  4. `git commit` to complete the merge.  
- **Abort**: `git merge --abort` to discard the merge and reset.  

#### **Merge Tools**  
| Tool | Setup | Best For |  
|------|--------|----------|  
| **VS Code** | `git config merge.tool vscode` | Beginners (visual diffs) |  
| **KDiff3** | `git config merge.tool kdiff3` | Complex conflicts (3-pane view) |  
| **vimdiff** | `git config merge.tool vimdiff` | CLI purists |  
| **Beyond Compare** | `git config merge.tool bc` | Professional teams |  

> 💡 **Pro Tip**: Use `git mergetool` to launch your configured tool. Always verify changes before committing!

---

### **IV. Rebasing**  
#### **What is Rebase?**  
- **Definition**: **Rewriting commit history** by moving/changing commits from one branch to another.  
- **Goal**: Create a **linear, clean history** (vs. merge’s explicit history).  
- **Golden Rule**: **Never rebase public/shared branches** (e.g., `main`). Only rebase **private branches**.  

#### **`git rebase` vs `git merge`**  
| **Rebase** | **Merge** |  
|------------|-----------|  
| Rewrites history (new commit SHAs) | Preserves history (original SHAs) |  
| Creates **linear history** | Creates **explicit merge commits** |  
| Risky for shared branches | Safe for all branches |  
| Preferred for **private feature branches** | Preferred for **public/release branches** |  

#### **Basic Rebase Workflow**  
```bash
git switch feature-auth
git rebase main  # Replays feature-auth commits ON TOP of main
```
- **Visualization**:  
  ```plaintext
  BEFORE:        A --- B --- C (main)
                         \
                          D --- E (feature-auth)
  
  AFTER REBASE:  A --- B --- C (main)
                                 \
                                  D' --- E' (feature-auth)
  ```  
  *(Note: `D'`/`E'` are **new commits** with different SHAs!)*  

#### **Interactive Rebase (`git rebase -i`)**  
- **Purpose**: **Edit, reorder, squash, or fix commits** before merging.  
- **Workflow**:  
  1. `git rebase -i HEAD~3` (rebase last 3 commits)  
  2. An editor opens showing commits:  
     ```plaintext
     pick a1b2c3 Add login button
     pick d4e5f6 Fix typo
     pick 098765 Update docs
     ```  
  3. **Commands**:  
     - `reword`: Edit commit message.  
     - `squash`: Combine commits (writes new message).  
     - `fixup`: Combine commits (discards message).  
     - `reorder`: Move commits up/down.  
     - `edit`: Pause to amend changes.  
  4. Save → Git processes changes step-by-step.  

> ⚠️ **Critical**: After rebasing, you **MUST force-push** (`git push -f`) since history changed. **Never do this on shared branches!**  

#### **When to Use Rebase vs Merge**  
| **Use Rebase** | **Use Merge** |  
|----------------|---------------|  
| Cleaning up **private feature branches** before PR | Merging into **public branches** (`main`, `develop`) |  
| Preparing a PR for review | When you **value historical accuracy** over neatness |  
| When no one else has the branch | When multiple people collaborate on a branch |  
| **Never** on shared branches | **Always** for shared branches |  

---

### **V. Advanced Branching Strategies**  
#### **Feature Branching**  
- **Workflow**:  
  1. Create branch from `main`: `git switch -c feat/login`  
  2. Develop feature → commit → push.  
  3. Open PR → review → **rebase** → merge to `main` (with `--no-ff`).  
- **Pros**: Isolates work, enables CI/CD per branch.  
- **Cons**: Long-lived branches risk merge hell.  

#### **Release Branching**  
- **Purpose**: Prepare a **production release** (e.g., v1.2).  
- **Workflow**:  
  1. Branch from `develop`: `git switch -c release/v1.2`  
  2. Finalize docs, version numbers, bug fixes.  
  3. Merge to `main` (tag as `v1.2`) **AND** back to `develop`.  
- **Why?**: Lets `develop` progress while stabilizing a release.  

#### **Hotfix Branching**  
- **Purpose**: **Urgent production fixes** (e.g., security patch).  
- **Workflow**:  
  1. Branch from `main`: `git switch -c hotfix/login-bug`  
  2. Fix → test → merge **directly to `main`** (tag version).  
  3. Merge **back to `develop`** to avoid re-introducing the bug.  

#### **Git Flow**  
- **Creator**: Vincent Driessen (2010).  
- **Branch Structure**:  
  - `main`: Production-ready code (tagged releases).  
  - `develop`: Integration branch for features.  
  - `feature/*`: Short-lived feature branches.  
  - `release/*`: Stabilization before release.  
  - `hotfix/*`: Critical production fixes.  
- **Pros**: Structured, clear for versioned software (e.g., desktop apps).  
- **Cons**: Complex for web apps with continuous deployment.  

#### **GitHub Flow**  
- **Creator**: GitHub (2011).  
- **Branch Structure**:  
  - `main`: Always deployable (production).  
  - `feature/*`: Short-lived branches → PR → **squash merge** → `main`.  
- **Key Principles**:  
  - No `develop` branch.  
  - Continuous deployment (CD) after merge to `main`.  
  - PRs are mandatory (with CI checks).  
- **Best For**: Web apps, SaaS products (e.g., GitHub, Netflix).  

#### **Trunk-Based Development (TBD)**  
- **Core Idea**: **All developers commit to `main` (trunk) daily** via short-lived branches (<1 day).  
- **How it Works**:  
  - Feature flags hide incomplete work.  
  - No long-lived branches → minimal merge conflicts.  
  - Heavy reliance on **automated testing** and CI/CD.  
- **Pros**: Enables continuous delivery, reduces integration pain.  
- **Cons**: Requires mature engineering culture (testing, small commits).  
- **Used By**: Google, Facebook, Amazon.  

---

### **VI. Critical Best Practices**  
1. **Never force-push (`-f`)** to shared branches (e.g., `main`, `develop`).  
2. **Rebase private branches** before PRs; **merge public branches**.  
3. **Resolve conflicts early**—don’t let branches diverge for weeks.  
4. **Delete merged branches** to avoid clutter (`git branch -d`).  
5. **Use descriptive branch names**: `feat/user-profile`, `fix/cart-bug`.  
6. **Squash feature commits** before merging (clean history).  

---

### **VII. When Things Go Wrong**  
| Problem | Solution |  
|---------|----------|  
| Accidentally committed to `main` | `git switch -c new-branch` → `git reset --hard HEAD~1` |  
| Messed up a rebase | `git reflog` → `git reset --hard <good-commit>` |  
| Lost unmerged work | `git fsck --lost-found` → recover dangling commits |  
| Merge conflict won’t resolve | `git merge --abort` → start over |  

---

### **Key Metaphors for Clarity**  
- **Branch**: A bookmark for your current work-in-progress.  
- **Merge**: A documented team decision ("We combined these efforts").  
- **Rebase**: Rewriting history as if you started your work *after* others finished.  
- **Conflict**: Two people editing the same sentence in a Google Doc.  

> ✅ **Final Pro Tip**: Branches are **cheap and easy**—use them for *everything*. Fear of branching is fear of Git itself. Master this, and you’ve mastered 80% of professional Git workflows.

---
