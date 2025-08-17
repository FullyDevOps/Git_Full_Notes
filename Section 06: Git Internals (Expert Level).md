### 🔄 1. The Reflog and Recovery

#### **What is `git reflog`?**
*   **Core Concept:** Git's **"safety net"**. It's a **log of all HEAD movements and reference updates** (branches, tags) *even if they aren't part of any branch history*. Unlike `git log` (which shows commit history), reflog tracks *your actions*.
*   **Why it Exists:** To recover from **accidental operations** (like `reset --hard`, `rebase`, branch deletion) that would otherwise make commits "disappear" from branch history. Git *doesn't immediately garbage collect* unreachable commits (default grace period: 30 days).
*   **How it Works:**
    *   Stored in `.git/logs/` (per branch) and `.git/logs/HEAD`.
    *   Records every time `HEAD` or a branch reference moves (e.g., commit, reset, merge, checkout).
    *   Entries look like: `c7a1c5e HEAD@{0}: reset: moving to HEAD~2` (SHA, reflog index, action, description).
*   **Key Command: `git reflog`**
    *   `git reflog`: Shows the **full reflog for HEAD** (most recent actions first).
    *   `git reflog show <branch>`: Shows reflog for a specific branch (e.g., `git reflog show main`).
    *   `git reflog --date=iso`: Shows timestamps in ISO format (critical for forensic recovery).
    *   `git reflog expire --expire=now --all`: **Dangerous!** Forces immediate garbage collection of unreachable commits (use only if you *know* you don't need recovery).

#### **Recovering Lost Commits**
*   **Scenario 1: Accidental `git reset --hard` (e.g., lost recent commits)**
    1.  Run `git reflog` to find the **SHA of the commit you were on BEFORE the reset** (look for entries like `HEAD@{1}: commit: ...`).
    2.  Reset back to it: `git reset --hard HEAD@{1}` OR `git reset --hard c7a1c5e` (using the SHA).
*   **Scenario 2: Deleted Branch (`git branch -D feature`)**
    1.  Run `git reflog` (or `git reflog show feature` if you remember the branch name).
    2.  Find the **last commit SHA on that branch** (e.g., `HEAD@{5}: checkout: moving from feature to main`).
    3.  Recreate the branch: `git branch feature <SHA>`.
*   **Scenario 3: Force Push Overwrite (`git push -f`)**
    1.  On the **local machine that did the force push**, run `git reflog`.
    2.  Find the **SHA of the commit that was the tip of the branch BEFORE the force push**.
    3.  Reset local branch: `git reset --hard <SHA>`.
    4.  Force push *again* to restore the remote: `git push -f origin main`.
    *   *Critical Note:* If others pulled the overwritten state, they'll need to reset locally too (coordinate carefully!).

#### **Detached HEAD State**
*   **What it is:** When `HEAD` points **directly to a commit** (SHA) instead of a branch reference. Common causes:
    *   `git checkout <commit-SHA>`
    *   `git checkout <tag>`
    *   `git checkout origin/main` (checking out a remote branch directly)
*   **Why it's a Problem:**
    *   **New commits are orphaned!** They aren't on *any* branch. If you switch branches, these commits become unreachable (but recoverable via reflog!).
    *   Git warns: `You are in 'detached HEAD' state. ... HEAD is now at ...`
*   **How to Fix/Use Safely:**
    *   **To keep changes:** Create a **new branch** from the detached HEAD: `git switch -c new-branch-name` (or `git checkout -b new-branch-name`).
    *   **To discard changes:** Simply check out a branch: `git switch main` (or `git checkout main`). The orphaned commits remain recoverable via reflog for ~30 days.
    *   **Intentional Use:** Valid for quick testing (e.g., `git checkout v2.0` to test a release tag). Always create a branch if you plan to make commits.

> 💡 **Reflog Golden Rule:** If you think you lost work, **RUN `git reflog` FIRST**. It's Git's undo history.

---

### 🧺 2. Stashing Changes

#### **Core Concept**
Temporarily **shelve (stash) uncommitted changes** (working directory + staging area) to clean your workspace *without* committing. Changes are saved to a **stack** (LIFO).

#### **Essential Commands**
*   **`git stash [save] ["message"]`**
    *   Saves **both tracked files** (modified/staged) and **untracked files** (`-u`/`--include-untracked`).
    *   **Untracked files:** Default stashes *only tracked files*. Use `-u` to include new files (e.g., `git stash -u "WIP: new feature draft"`).
    *   **Ignored files:** Use `-a`/`--all` to also stash ignored files (rarely needed).
    *   *Message is crucial for managing multiple stashes!*
*   **`git stash list`**
    *   Shows all stashes: `stash@{0}: On main: WIP...`, `stash@{1}: ...`.
*   **`git stash pop [stash@{n}]`**
    *   **Applies** the top stash (`stash@{0}`) **and deletes it** from the stack.
    *   **Conflict Risk:** If changes conflict with current state, Git stops and marks conflicts (resolve like a merge).
    *   Specify stash: `git stash pop stash@{2}`.
*   **`git stash apply [stash@{n}]`**
    *   **Applies** the stash **without deleting it** from the stack. Safe for testing.
    *   Use when you might need to re-apply or apply to multiple branches.
*   **`git stash drop [stash@{n}]`**
    *   Deletes a specific stash (e.g., `git stash drop stash@{1}`).
*   **`git stash clear`**
    *   **DELETES ALL STASHES** (irreversible!). Use with extreme caution.

#### **Advanced Stash Management**
*   **Multiple Stashes:** Treat the stash stack like a to-do list. Use descriptive messages!
*   **View Stash Contents:**
    *   `git stash show [stash@{n}]`: Brief summary (like `git diff --stat`).
    *   `git stash show -p [stash@{n}]`: Full patch (like `git diff`).
*   **Create Branch from Stash:** `git stash branch new-branch stash@{1}`. Creates branch at the commit *when the stash was made*, applies stash, and drops it. **Best practice for complex stashes.**
*   **Stash Only Staged Changes:** `git stash push -m "Staged only" -- <path>` (stashes *only* staged changes for `<path>`; unstaged changes remain).

> ⚠️ **Critical Pitfalls:**
> *   Stashes **DO NOT** include untracked files by default (`-u` required).
> *   `pop` **deletes** the stash; `apply` does not. Prefer `apply` first to test.
> *   Stashes are **local only** (not pushed to remote).
> *   Stash stack is **per-repository**, not per-branch.

---

### 🍒 3. Cherry-Picking

#### **Core Concept**
Apply the **changes introduced by *one specific commit*** from *any branch* onto your **current branch**. Creates a **new commit** with the same changes (but a new SHA).

*   **Command:** `git cherry-pick <commit-SHA>` (e.g., `git cherry-pick abc123`)
*   **How it Works:**
    1.  Git calculates the **diff** between `<commit-SHA>` and its parent.
    2.  Tries to **apply that diff** to your current `HEAD`.
    3.  Creates a **new commit** with the applied changes (author/date preserved by default).

#### **Use Cases (When to Use)**
1.  **Hotfix Propagation:** Fix a bug on `main`, then cherry-pick the fix commit onto `release/v1.2`.
2.  **Selective Feature Integration:** Pull one critical feature commit from a long-running branch into `main` before the whole branch is ready.
3.  **Undoing a Bad Merge:** If a merge introduced a single bad commit, revert *that specific commit* via cherry-pick (often easier than full revert).
4.  **Moving Commits Between Branches:** When rebase isn't suitable (e.g., public branch history shouldn't change).

#### **Risks & Critical Considerations**
*   **🚫 History Rewriting (Implicit):** Creates a **new commit** with a different SHA. **NEVER cherry-pick commits that are already in a shared/public branch history** (causes duplicate commits, confusion, merge conflicts later). Only use on private branches or for hotfixes *forward*.
*   **🧩 Merge Conflicts:** Highly likely if the codebase has diverged since `<commit-SHA>` was made. You **must resolve conflicts** manually during the cherry-pick.
*   **Context Loss:** The commit might depend on other changes *not* cherry-picked. Test thoroughly!
*   **Authorship:** By default, the **cherry-picked commit retains the original author** (but committer is you). Use `--no-commit` to stage changes without committing (review before final commit).
*   **Flags:**
    *   `-n`/`--no-commit`: Stage changes but don't commit (review/modify first).
    *   `-x`: Adds `(cherry picked from commit ...)` to the commit message (recommended for transparency in shared repos).
    *   `-e`/`--edit`: Edit the commit message before finalizing.
    *   `-m <parent-number>`: For cherry-picking **merge commits** (specify mainline parent, usually `-m 1`).

> 💡 **Golden Rule:** Cherry-pick is for **isolated, self-contained changes**. If changes depend on multiple commits, consider `rebase -i` or merging the whole branch instead.

---

### 🏷️ 4. Tagging

#### **Two Types of Tags**
| Feature          | Lightweight Tag                     | Annotated Tag                                  |
| :--------------- | :---------------------------------- | :--------------------------------------------- |
| **What it is**   | Simple pointer to a commit (SHA)    | **Full Git object** (like a commit)            |
| **Created with** | `git tag <tagname>`                 | `git tag -a <tagname> -m "Message"`            |
| **Stores**       | Commit SHA only                     | Tagger name/email, date, **message**, GPG sig |
| **Recommended**  | ❌ (Internal use only)              | ✅ (For releases, public references)           |
| **Verification** | No                                  | ✅ (Can be GPG signed: `git tag -s v1.0.1`)    |

#### **Creating Tags**
*   **Annotated (Best Practice):**
    ```bash
    git tag -a v2.1.0 -m "Stable release for client X"  # Tag current HEAD
    git tag -a v2.0.0 abc123                          # Tag specific commit
    ```
*   **Signed (Annotated + GPG):**
    ```bash
    git tag -s v1.2.3 -m "Signed release"  # Uses your GPG key
    ```
*   **Lightweight (Rarely Needed):**
    ```bash
    git tag experimental-branch-tip  # Points to current HEAD
    ```

#### **Managing Tags**
*   **List Tags:** `git tag` (alphabetical), `git tag -l "v2.*"` (pattern), `git tag -n` (with message snippet).
*   **View Tag Details:** `git show v2.1.0` (shows tag info + commit).
*   **Delete Local Tag:** `git tag -d v0.9.0`.
*   **Checkout a Tag:** `git switch v1.0` (detached HEAD - create a branch if needed!).

#### **Pushing Tags to Remote**
*   **Push Specific Tag:** `git push origin v2.1.0`
*   **Push ALL Tags:** `git push origin --tags` (⚠️ Use cautiously! Pushes *all* local tags)
*   **Delete Remote Tag:**
    ```bash
    git push origin --delete v0.9.0  # Modern syntax
    git push origin :refs/tags/v0.9.0 # Legacy syntax
    ```

#### **Using Tags for Releases**
1.  **Stable Reference:** Tags are **immutable pointers** to a specific code state (unlike branches which move).
2.  **Release Workflow:**
    *   Finish features/fixes on `main`.
    *   `git tag -a v3.0.0 -m "Production release"`
    *   `git push origin v3.0.0`
    *   CI/CD systems deploy from the tag (e.g., `git checkout v3.0.0`).
3.  **Hotfixes:** Fix on `main`, tag `v3.0.1`, deploy from tag.
4.  **Semantic Versioning:** Tags like `v1.2.3` are standard practice.

> 💡 **Pro Tip:** Always use **annotated tags** for releases. Lightweight tags are fragile (easily overwritten, no metadata).

---

### 🔗 5. Submodules and Subtrees

#### **Submodules: Git Repos Inside Git Repos**
*   **What it is:** A **.gitmodules file** (tracks submodule URL/path) + a **gitlink** (SHA pointer in parent repo's tree) to a **specific commit** in the submodule repo.
*   **Core Idea:** Parent repo **references a fixed state** of another repo. Submodule is a **separate Git repo** nested inside the parent working directory.

*   **Adding a Submodule:**
    ```bash
    git submodule add -b main https://github.com/user/dep.git vendor/dep
    # -b main: Tracks 'main' branch (optional, but recommended)
    # Creates .gitmodules + initializes submodule at vendor/dep
    git commit -m "Add dep submodule"
    ```
*   **Cloning a Repo with Submodules:**
    ```bash
    git clone --recurse-submodules https://github.com/user/project.git
    # OR
    git clone https://github.com/user/project.git
    cd project
    git submodule update --init --recursive
    ```
*   **Updating Submodules:**
    *   **Parent Repo Perspective:** Submodule is at a specific commit. To update:
        ```bash
        cd vendor/dep
        git fetch
        git checkout v2.0  # Or git merge origin/main
        cd ..
        git add vendor/dep  # Records *new* submodule commit in parent
        git commit -m "Update dep to v2.0"
        ```
    *   **Bulk Update (All Submodules):** `git submodule update --remote --merge` (fetches latest for tracked branch, merges).
*   **Removing a Submodule:**
    1.  `git rm -f vendor/dep` (removes from index + working dir)
    2.  Remove section from `.gitmodules`
    3.  Remove `vendor/dep/.git` (if exists)
    4.  `git commit -m "Remove dep submodule"`

*   **Pros of Submodules:**
    *   Explicit version control (points to specific commit).
    *   Independent history for dependencies.
    *   Saves disk space (shared objects if same submodule used elsewhere).
*   **Cons of Submodules:**
    *   **Complex Workflow:** Nested repos, extra commands (`submodule update`).
    *   **"Detached HEAD" Risk:** Submodules checkout specific commits by default.
    *   **Brittleness:** `.git` folder inside submodule can cause issues (e.g., accidental commits).
    *   **Shallow Clones:** Harder to manage (requires `--depth` flags).
    *   **Contributor Friction:** New users often struggle.

#### **Subtree Merging: Alternative Approach**
*   **What it is:** **Merges the *entire history* of another repo as a subdirectory** into your main repo's history. No separate `.git` folder.
*   **How it Works:**
    1.  Add the dependency repo as a **remote**.
    2.  `git read-tree` or `git merge -s ours` to combine histories.
    3.  Dependency lives as a normal subdirectory in your repo.

*   **Adding a Subtree:**
    ```bash
    git remote add dep-remote https://github.com/user/dep.git
    git fetch dep-remote
    git merge -s ours --no-commit dep-remote/main  # Merge history, but don't commit changes
    git read-tree --prefix=vendor/dep/ -u dep-remote/main  # Bring files into /vendor/dep
    git commit -m "Merge dep as subtree"
    ```
    *(Simpler with `git subtree` plugin - see below)*
*   **Updating a Subtree:**
    ```bash
    git fetch dep-remote
    git merge -s subtree dep-remote/main  # OR use plugin
    ```
*   **Pushing Changes Back (Rare):**
    ```bash
    git subtree push --prefix=vendor/dep dep-remote main
    ```

*   **Pros of Subtrees:**
    *   **Simpler Workflow:** No nested repos. Standard Git commands work.
    *   **No Detached HEAD:** All code is part of main repo history.
    *   **Easier Cloning:** `git clone` gets everything immediately.
    *   **Better for Contributors:** No special submodule setup.
*   **Cons of Subtrees:**
    *   **Bloats Main History:** Full dependency history is in your repo.
    *   **Complex History:** Merge commits can be messy.
    *   **Harder to Update:** Requires careful subtree merge/push.
    *   **No Explicit Versioning:** Less clear which *exact* dependency commit you're on (check merge commit message).

*   **The `git subtree` Plugin (Highly Recommended):**
    *   Simplifies subtree commands: `git subtree add`, `git subtree pull`, `git subtree push`.
    *   Install: Usually bundled with Git (`git subtree --help` to check).
    *   **Example Workflow:**
        ```bash
        # Add
        git subtree add --prefix=vendor/dep https://github.com/user/dep.git main --squash
        # Update
        git subtree pull --prefix=vendor/dep https://github.com/user/dep.git main --squash
        # Push changes back (if you maintain dep)
        git subtree push --prefix=vendor/dep https://github.com/user/dep.git main
        ```
    *   `--squash`: Combines all dependency commits into one (cleaner history).

> 💡 **When to Choose:**
> *   **Submodules:** You need **strict version control** of dependencies, work with large binaries, or the dependency is actively developed *alongside* your project (e.g., microservices).
> *   **Subtrees (with plugin):** You want **simplicity for users**, the dependency is stable, or you rarely update it. Best for libraries where you don't need the full dependency history.

---

### ✉️ 6. Patches and Email-Based Workflows

#### **Core Concept**
Exchange changes as **text files (patches)** via email (common in open-source projects like the Linux kernel). Uses **plain text diffs** instead of Git remotes.

#### **Creating Patches: `git format-patch`**
*   **What it Does:** Generates **1+ `.patch` files** (one per commit) containing:
    *   Commit metadata (SHA, author, date, subject)
    *   Full commit message
    *   Patch (diff) in standard unified diff format (`git diff -p`)
*   **Basic Usage:**
    ```bash
    git format-patch -1 abc123        # Creates 0001-<subject>.patch for commit abc123
    git format-patch main..feature    # Creates patches for all commits in feature not in main
    ```
*   **Key Flags:**
    *   `-n`: Number of patches (e.g., `-3` for last 3 commits).
    *   `--start-number <N>`: Start numbering at N (e.g., for v2 of a patch series).
    *   `--subject-prefix "RFC PATCH"`: Customize email subject prefix (e.g., `[PATCH v2]`).
    *   `--cover-letter`: Generates a `0000-cover-letter.patch` summarizing the series.
    *   `--to "dev-list@example.com" --cc "maintainer@example.com"`: Pre-fills email headers.
    *   `--output-directory out/`: Write patches to a dir.

#### **Applying Patches: `git am` (Apply Mailbox)**
*   **What it Does:** Applies patches **created by `format-patch`** (or mbox format). Preserves authorship, commit message, and date.
*   **Basic Usage:**
    ```bash
    git am 0001-fix-login.patch       # Apply single patch
    git am *.patch                    # Apply all patches in order
    ```
*   **Workflow:**
    1.  Receive patch email (or save `.patch` files).
    2.  `git am < patch-file.patch` OR `git am *.patch`
    3.  **Resolve Conflicts (if any):** Git stops at first conflict. Fix files, then `git add <fixed-files>`, then `git am --continue`.
    4.  `git am --skip`: Skip a problematic patch.
    5.  `git am --abort`: Cancel the entire apply operation.
*   **Why `am` > `apply`:**
    *   `git apply`: Only applies the *diff* (loses authorship/date/message).
    *   `git am`: Applies the **full commit** (metadata + diff) = creates a real Git commit.

#### **Linux Kernel-Style Workflow (Real-World Example)**
1.  **Developer:**
    *   Makes commits on a branch.
    *   `git format-patch -s -o outgoing/ main` (`-s` adds "Signed-off-by").
    *   Sends patches via `git send-email` (or attaches to email).
2.  **Maintainer:**
    *   Receives patch emails.
    *   Saves patches to disk (or uses `git am` directly from email client).
    *   `git am --signoff *.patch` (applies patches, adds maintainer signoff).
    *   Reviews, tests, potentially edits commits (`git rebase -i`).
    *   Pushes to main repo.
3.  **Key Principles:**
    *   **Patches are the source of truth**, not shared remotes.
    *   **Sign-off (`-s`)**: Proves contributor agreement (DCO - Developer Certificate of Origin).
    *   **Patch Series:** Numbered patches (`0001-`, `0002-`) with a cover letter.
    *   **Iteration:** `v2`, `v3` in subject line for revised patches.
    *   **No Force Push:** History is immutable once patches are sent.

#### **Why Use This Workflow?**
*   **Open Source Collaboration:** Works without shared writable remotes (anyone can submit).
*   **Audit Trail:** Patches are archived in mailing lists (public record).
*   **Maintainer Control:** Maintainer curates patches before merging.
*   **Offline Work:** Patches can be reviewed/applied without network.
*   **Legacy Systems:** Works where Git remotes aren't feasible.

> 💡 **Critical Note:** `git send-email` is the companion tool for sending patches (configures SMTP, handles threading). `git format-patch` + `git am` is the core exchange mechanism.

---

### ✅ Summary: Key Takeaways for Your Reference

| Topic               | Critical Insight                                                                 | Must-Remember Command                          |
| :------------------ | :------------------------------------------------------------------------------- | :--------------------------------------------- |
| **Reflog**          | Your **ONLY** hope for lost commits; tracks *your actions*, not just history.    | `git reflog` → `git reset --hard HEAD@{n}`     |
| **Stashing**        | **`-u` for untracked files!** `pop` deletes stash; `apply` does not.             | `git stash -u "msg"` → `git stash apply`       |
| **Cherry-Pick**     | **NEVER** on public history; creates duplicate commits. Use `-x` for transparency. | `git cherry-pick -x abc123`                    |
| **Tagging**         | **ALWAYS use `-a` for releases.** Lightweight tags are dangerous.                | `git tag -a v1.2.3 -m "Msg"` → `git push origin v1.2.3` |
| **Submodules**      | **Complex but precise.** Remember `--recurse-submodules` on clone.               | `git submodule update --init --recursive`      |
| **Subtrees**        | **Simpler for users.** Use `git subtree` plugin. `--squash` keeps history clean. | `git subtree pull --prefix=lib/dep ...`        |
| **Patches**         | `format-patch` → `am` preserves **full commits**; `apply` does not.              | `git format-patch -1` → `git am < patch.patch` |
