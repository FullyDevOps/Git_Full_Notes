### **1. Interactive Rebase Deep Dive**
*   **What it is:** A powerful command (`git rebase -i`) that lets you **interactively rewrite commit history** *before* it's shared publicly. It opens an editor listing commits to modify.
*   **Why it matters:** Essential for maintaining clean, logical, and professional commit histories before pushing to shared branches (like `main`/`develop`). Avoids polluting history with "WIP" or broken intermediate states.
*   **Core Commands & Workflow:**
    1.  **Initiate:** `git rebase -i <base-commit>` (e.g., `git rebase -i HEAD~3` to edit last 3 commits)
    2.  **Editor Opens:** Lists commits from oldest (top) to newest (bottom) with command verbs:
        *   `pick`: Keep commit as-is (default).
        *   `reword (r)`: Keep commit *changes* but edit the *commit message*.
            *   *How:* Change `pick` to `reword` next to commit. Save/exit editor -> New editor opens for message.
        *   `edit (e)`: Pause rebase *at this commit*. Allows amending the commit (add/remove files, change message) or splitting it.
            *   *How:* Change `pick` to `edit`. Save/exit -> Rebase pauses. Make changes -> `git add .` -> `git commit --amend` -> `git rebase --continue`.
        *   `squash (s)`: Combine this commit's *changes* into the **previous** commit. Prompts for a new combined message.
            *   *How:* Change `pick` of commit *to be squashed* to `squash`. Save/exit -> Editor opens for combined message.
        *   `fixup (f)`: Like `squash`, but **discards the squashed commit's message**. Uses the previous commit's message.
            *   *How:* Change `pick` to `fixup`. Save/exit -> No message prompt, rebase continues silently.
        *   `drop (d)`: **Permanently remove** this commit from history.
            *   *How:* Change `pick` to `drop`. Save/exit -> Commit is gone. **WARNING:** Irreversible if history is shared!
    3.  **Save & Exit:** Git processes commands in order. May pause for `reword`/`edit`/`squash` messages.
    4.  **Resolve Conflicts (if any):** If conflicts occur during rebase, fix them -> `git add <resolved-files>` -> `git rebase --continue`.
    5.  **Abort (if needed):** `git rebase --abort` (resets everything to pre-rebase state).

*   **Critical Nuances & Gotchas:**
    *   **NEVER rebase public/shared branch history** (e.g., `main` after others have pulled). Only rebase *your own* local commits before pushing.
    *   **Order Matters:** Commands are executed top-to-bottom. `squash`/`fixup` *must* be placed *below* the commit they merge into.
    *   **Amending vs. Editing:** `--amend` changes the *current* commit. `rebase -i` with `edit` lets you modify *any* past commit in the sequence.
    *   **Splitting a Commit:** Use `edit` on the commit -> `git reset HEAD~` (unstages changes) -> Stage/commit subsets individually -> `git rebase --continue`.
    *   **Rewriting History = New Commits:** Every modified commit gets a *new SHA-1 hash*. This breaks existing references to the old commits.

*   **Fixup and Autosquash**
    *   **`fixup`:** As described above (`f` verb). Merges changes silently, discarding message.
    *   **`autosquash`:** Automates the `fixup` process for *later* commits.
        *   **Workflow:**
            1.  Make a "fixup" commit targeting an *older* commit: `git commit --fixup=<old-commit-SHA>`
                *   Message auto-formatted: `fixup! <original-commit-message>`
            2.  Start interactive rebase: `git rebase -i --autosquash <base-commit>`
        *   **What Happens:** Git automatically:
            1.  Reorders the `fixup! ...` commit *directly below* the target commit it references.
            2.  Changes its verb from `pick` to `fixup`.
        *   **Why it's Awesome:** No manual reordering/searching in the rebase todo list. Perfect for quick follow-up fixes during development.
        *   **Enable Globally (Optional):** `git config --global rebase.autoSquash true` (Use `--autosquash` flag still needed, but config enables the feature).

*   **Rebasing Across Branches**
    *   **What it is:** Using `git rebase` to integrate changes from one branch *onto* another branch, creating a linear history.
    *   **Workflow (Integrating `feature` into `main`):**
        1.  `git checkout feature`
        2.  `git rebase main` (Replays `feature` commits *on top of* current `main`)
        3.  `git checkout main`
        4.  `git merge feature` (Fast-forward merge - now linear!)
    *   **vs. Merging:**
        *   **Rebase:** Creates linear history (cleaner, easier `git log --oneline`), *rewrites* `feature`'s commit SHAs.
        *   **Merge:** Preserves exact history/time of branching (non-linear), creates a merge commit, *does not rewrite* SHAs.
    *   **Golden Rule:** Only rebase branches you *own* (local/private branches). Never rebase branches others are using (`main`, `develop`, shared feature branches).
    *   **When to Use:** Before merging a *private* feature branch into `main` to keep history clean. Essential for "merge request" workflows where clean history is required.

---

### **2. Reflog Recovery Scenarios**
*   **What it is:** The **Reflog (Reference Log)** is Git's *local safety net*. It records **every change** to *any* branch or `HEAD` reference *on your machine*, for ~90 days (configurable). Think of it as a "timeline of where your tips have been".
*   **Why it matters:** Your **ONLY hope** for recovering commits *after* they've been lost via:
    *   `git reset --hard` (accidental or intentional)
    *   `git commit --amend` (overwriting the last commit)
    *   `git rebase` (messing up and aborting)
    *   **Deleted branches** (local branches, *not* remote)
    *   **Force-pushed changes** (if you had the old state locally)
*   **How to Use:**
    *   **View Reflog:** `git reflog` (Shows all reference movements)
        *   `git reflog show <branch>` (e.g., `git reflog show main`)
        *   `git log -g` (Shows reflog entries as a log)
    *   **Key Columns:** `HEAD@{n}` (Relative position), `SHA`, `Action` (`commit`, `reset`, `rebase`, `checkout`, `amend`), `Message`
*   **Recovery Scenarios:**
    *   **Scenario 1: Recovering a Deleted Local Branch**
        1.  `git reflog` (Find the last SHA where the branch tip was, e.g., `abc123 HEAD@{5}: checkout: moving from feature-x to main`)
        2.  `git branch <lost-branch-name> abc123` (Recreates branch pointing to that SHA)
    *   **Scenario 2: Undoing a `git reset --hard`**
        1.  `git reflog` (Find the SHA *before* the reset, e.g., `def456 HEAD@{1}: reset: moving to HEAD~3`)
        2.  `git reset --hard def456` (Moves HEAD/branch back to that SHA)
    *   **Scenario 3: Undoing a Force Push (`git push -f`)**
        *   **Crucial:** Only works if *you* had the pre-force-push state *locally* before pushing.
        1.  `git reflog` (Find the SHA of the branch tip *before* the force push, e.g., `ghi789 main@{2}: commit: My good work`)
        2.  `git reset --hard ghi789` (Resets your *local* `main` to the old state)
        3.  `git push -f origin main` (Force pushes the *corrected* history back up. **Only do this if you're sure & it's your branch!**)
    *   **Scenario 4: Time-Travel Debugging (Finding a Bug Introduction)**
        *   **Reflog's Role:** Helps find *specific historical states* quickly.
        1.  Use `git reflog` to identify a known-good SHA (`good-sha`) and a known-bad SHA (`bad-sha`).
        2.  Run `git bisect start`
        3.  `git bisect bad <bad-sha>` (Current state is bad)
        4.  `git bisect good <good-sha>` (Identify a known good state)
        5.  Git checks out a midpoint commit. Test: Is the bug present?
            *   `git bisect good` (Bug NOT present here)
            *   `git bisect bad` (Bug IS present here)
        6.  Repeat step 5 until bisect identifies the *first* bad commit.
        *   **Reflog Connection:** If you lose your place during bisect, `git reflog` shows all the bisect steps (`bisect: ...`). You can `git reset --hard` to a specific bisect state.

*   **Critical Limitations:**
    *   **LOCAL ONLY:** Reflog is *per-repository, per-machine*. It does **NOT** recover commits lost *only* on the remote server (unless you had them locally).
    *   **Garbage Collection:** Old reflog entries expire (~90 days default, `git config gc.reflogExpire`). Run `git reflog expire --all --expire=now` to clean *now* (rarely needed).
    *   **Not for Remote Recovery:** If you `git push -f` and lose commits *and no one else has them locally*, they are **gone forever**. Reflog only helps *your local copy*.

---

### **3. Sparse Checkouts**
*   **What it is:** A feature allowing you to **check out only a subset of files/directories** from your Git repository into your working directory, even though the *entire history* is still in `.git`.
*   **Why it matters:** **Essential for large monorepos** (single repo with many projects/services). Solves:
    *   Slow `git status`/`git add` due to massive working trees.
    *   IDEs indexing irrelevant files, slowing down.
    *   Disk space constraints (though `.git` is still full size).
    *   Focusing on only the relevant part of the codebase.
*   **How it Works (Core Concept):** Git tracks which files are "sparse" (excluded from working copy) using a special pattern file (`sparse-checkout`). The index (staging area) still knows about *all* files, but only "included" files are physically present on disk.
*   **Setup & Usage (Modern Method - `git sparse-checkout`):**
    1.  **Initialize Sparse Checkout Mode:**
        ```bash
        git clone --filter=blob:none --no-checkout <repo-url>  # Optional but recommended for huge repos
        cd <repo>
        git sparse-checkout init --cone  # Enables "cone mode" (faster, simpler patterns)
        ```
        *   `--filter=blob:none`: (Optional) Skips downloading file *contents* during clone (only gets directory structure/history metadata). Requires `git config --global feature.multipleWorktrees true` (usually default).
        *   `--cone`: Uses optimized pattern matching (only supports prefix patterns like `src/`, `projects/foo/`). **Highly recommended.**
    2.  **Define What to Checkout:**
        ```bash
        git sparse-checkout set path/to/dir1 path/to/file2 projects/awesome-app/
        ```
        *   Replaces the current sparse pattern set with these paths.
        *   Uses **cone mode patterns** (directories only, no complex globs).
    3.  **Checkout the Defined Content:**
        ```bash
        git checkout  # Populates working directory with only the defined paths
        ```
    4.  **Modify Patterns Later:**
        ```bash
        git sparse-checkout add path/to/new/dir  # Add more paths
        git sparse-checkout set path1 path2     # Replace all patterns
        git sparse-checkout disable             # Disable sparse mode (check out everything)
        ```
*   **Use Cases (Monorepo Focus):**
    *   **Frontend Dev:** Only check out `web/` and `shared/ui/` directories.
    *   **Backend Dev:** Only check out `api/` and `shared/core/`.
    *   **Mobile Dev:** Only check out `mobile/ios/` and `mobile/android/`.
    *   **CI/CD Job:** Only check out the specific service directory needed for the build/test step.
*   **Key Considerations:**
    *   **`.git` Size:** The `.git` directory still contains the *full history* of the entire repo. Sparse checkout *only* affects the working directory.
    *   **Cone Mode vs. Classic:** **ALWAYS USE `--cone` (`git sparse-checkout init --cone`)**. Classic mode (complex patterns) is slow and error-prone. Cone mode is fast and uses simple directory prefixes.
    *   **`git status`:** Will show "deleted" for files *outside* your sparse patterns (they exist in Git but not on disk). This is **normal and expected**. Don't try to "add" them back!
    *   **Not for Partial Clones:** Sparse checkout works *with* the full local repo history. For truly partial clones (only recent history), combine with **Shallow Clones** (see below).
    *   **IDE Support:** Modern IDEs (VSCode, IntelliJ) handle sparse checkouts well. Older tools might get confused by "missing" files Git knows about.

---

### **4. Worktrees**
*   **What it is:** The `git worktree` command allows you to have **multiple, independent working directories** attached to a *single* Git repository (`.git` directory). Each worktree has its own branch checked out.
*   **Why it matters:** Solves the problem of **needing to context-switch between branches without stashing**. Crucial for:
    *   **Parallel Feature Development:** Work on `feature/login` in one window, `feature/search` in another, simultaneously.
    *   **Quick Bug Fixes:** While deep in a feature (`feature/checkout`), instantly create a new worktree for `hotfix/payment`, fix it, commit, and switch back without disturbing your main work.
    *   **Testing PRs:** Check out a colleague's PR branch in a separate worktree for testing without leaving your current branch.
    *   **Avoiding Stash Hell:** Eliminates constant stashing/unstashing when switching contexts.
*   **How it Works:**
    *   The main repo (`.git` directory) is the **"primary worktree"** (usually your original checkout).
    *   Each `git worktree add` creates a new **"linked worktree"**:
        *   Has its *own* working directory (a separate folder on disk).
        *   Has its *own* `HEAD`, index (staging area), and local files.
        *   Shares the *same* underlying `.git` object database and refs (branches/tags).
    *   **Cannot** have two worktrees on the *same* branch simultaneously (Git prevents this).
*   **Core Commands:**
    *   **Create a New Worktree:**
        ```bash
        git worktree add [-b <new-branch-name>] <path/to/new/worktree> [<start-point>]
        ```
        *   `-b new-feature`: Creates *and* checks out a new branch `new-feature` starting from `<start-point>` (e.g., `origin/main`).
        *   `<path/to/new/worktree>`: Absolute or relative path to the *new directory* (must not exist).
        *   `<start-point>`: Commit/branch to start from (default: current branch HEAD).
        *   *Example:* `git worktree add -b hotfix/login-bug ../hotfixes origin/main`
    *   **List Worktrees:**
        ```bash
        git worktree list
        ```
        Shows all linked worktrees, their paths, current branch, and if "bare" (primary worktree is usually listed as `bare`).
    *   **Delete a Worktree (Safely):**
        ```bash
        git worktree remove <path/to/worktree> [--force]
        ```
        *   **Crucial:** *Always* use `git worktree remove` to delete the directory. **Do NOT `rm -rf`!**
        *   Git checks for uncommitted changes/dirty state. Use `--force` *only* if you're sure (loses uncommitted work).
        *   This cleans up Git's internal tracking of the worktree.
    *   **Prune Stale Entries:**
        ```bash
        git worktree prune
        ```
        Cleans up metadata for worktrees that were deleted *without* using `git worktree remove` (e.g., `rm -rf`'d the dir). Run if `git worktree list` shows dead entries.
*   **Managing Parallel Work:**
    *   **Switching is Instant:** `cd ../hotfixes` -> you're on `hotfix/login-bug`. `cd ../main-work` -> you're back on `feature/checkout`.
    *   **Independent State:** Each worktree has its own uncommitted changes, stash, etc. No conflict.
    *   **Branch Management:** Creating/deleting branches in *one* worktree is visible in *all* worktrees (since they share `.git/refs`).
    *   **Locking:** Git locks a worktree's branch while it's checked out elsewhere. Prevents accidental simultaneous edits.
*   **Critical Gotchas:**
    *   **Don't Nest Worktrees:** Avoid creating a worktree *inside* another worktree's directory. Git can get confused.
    *   **Don't Move Worktree Directories Manually:** Use `git worktree move <old-path> <new-path>` if you *must* relocate one.
    *   **Primary Worktree:** The original clone directory is the primary worktree. You can't delete it while linked worktrees exist (Git will warn).
    *   **Disk Space:** Each worktree has a full working copy (files on disk). `.git` is shared, so objects aren't duplicated, but working files are.
    *   **Hooks:** Hooks (like `pre-commit`) in `.git/hooks` apply to *all* worktrees. For worktree-specific hooks, use `git config core.hooksPath <path>` *within* that worktree.

---

### **5. Shallow Clones and Cloning Optimization**
*   **What it is:** Techniques to **reduce the amount of history/data** fetched during `git clone`, significantly speeding up the process for large repositories or CI/CD pipelines where full history isn't needed.
*   **Why it matters:**
    *   **CI/CD Speed:** Cloning a massive repo with decades of history can take *minutes*. Shallow clones take *seconds*.
    *   **Bandwidth Savings:** Critical for limited bandwidth or large teams.
    *   **Disk Space (Temporarily):** Smaller initial clone size (though deepening later adds data).
*   **Core Techniques:**
    *   **`git clone --depth <n>` (Shallow Clone):**
        *   Fetches *only* the latest `<n>` commits from the default branch (usually `main`/`master`).
        *   *Example:* `git clone --depth 1 https://github.com/user/large-repo.git`
        *   **Pros:** Extremely fast initial clone. Perfect for CI jobs that only need the latest code to build/test.
        *   **Cons:**
            *   **No Full History:** `git log` only shows `<n>` commits. `git blame` might be inaccurate near the shallow boundary.
            *   **Limited Operations:** Cannot `git log` beyond depth, `git blame` deep history, `git bisect` over full history, or `git rebase` effectively.
            *   **Detached HEAD:** Often starts in detached HEAD state (points to latest commit SHA, not a branch). Fix: `git switch -c main origin/main` (if `main` is default).
        *   **Deepening Later (If Needed):**
            *   `git fetch --depth=<new-depth>` (e.g., `--depth=100`)
            *   `git fetch --deepen=<n>` (Adds `n` more commits)
            *   `git fetch --unshallow` (Fetches *all* history - slow!)
    *   **`git clone --single-branch --branch <branch-name>`:**
        *   Fetches *only* the history for *one specific branch*, ignoring all other branches and their history.
        *   *Example:* `git clone --single-branch --branch feature/new-ui https://github.com/user/large-repo.git`
        *   **Pros:** Huge savings if the repo has many long-lived branches. Essential when combined with `--depth` for CI.
        *   **Cons:** Cannot see or fetch other branches without modifying the remote config later (`git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"` then `git fetch`).
        *   **Combine with `--depth`:** `git clone --depth 1 --single-branch --branch main https://github.com/user/repo.git` (Most common CI pattern).
*   **Use Cases in CI/CD:**
    *   **Standard Build Job:** `git clone --depth 1 --single-branch --branch $CI_COMMIT_REF_NAME .` (Get *only* the exact commit for the current branch/pipeline).
    *   **Jobs Needing Limited History:** `git clone --depth 50 --single-branch ...` (For changelog generation, basic `git blame`).
    *   **Jobs Needing Full History (Rare in CI):** Avoid shallow clones. Might need `--no-single-branch` and sufficient depth or `--unshallow`.
*   **Critical Considerations & Gotchas:**
    *   **Not for Development:** Shallow clones are **unsuitable for active development** where you need full history (blame, bisect, rebase, understanding context). Use full clones locally.
    *   **CI Pipeline Design:** Structure pipelines so *only* jobs that *truly* only need the latest commit use shallow clones. Jobs doing versioning, changelogs, or complex history analysis need more depth.
    *   **Tag Fetching:** `--depth` *does not* fetch tags by default. Use `--tags` *with* `--depth` if you need recent tags (e.g., `--depth 1 --tags`).
    *   **Submodules:** Shallow clones can complicate submodule initialization. Often requires `--recurse-submodules --depth 1` or similar.
    *   **Git Version:** Ensure your CI runners have a reasonably modern Git (>= 2.27 for best shallow clone reliability, but most features work since ~1.9).
    *   **`fetch` vs `pull`:** `git pull` in a shallow clone might fail if it tries to update beyond the shallow depth. Prefer explicit `git fetch --depth=N` followed by `git reset --hard origin/<branch>` in CI.

---

### **When to Use What: Quick Decision Guide**

| Scenario                                      | Recommended Tool(s)                          | Why                                                                 |
| :-------------------------------------------- | :------------------------------------------- | :------------------------------------------------------------------ |
| Cleaning up *your own* local commits before PR | **Interactive Rebase** (`squash`, `fixup`)   | Creates clean, logical history for reviewers.                       |
| Fixing a typo in a commit message *you just made* | `git commit --amend`                         | Quickest way for the *very last* commit.                            |
| Recovering a branch you accidentally deleted  | **Reflog**                                   | Only local safety net for lost references.                          |
| Working on 2 features at once                 | **Worktrees**                                | No stashing, instant context switch, independent state.             |
| Developing in a massive monorepo              | **Sparse Checkout** (with `--cone`)          | Only load relevant code, speed up IDE/Git operations.               |
| CI Build Job (only needs latest code)         | **Shallow Clone** (`--depth 1 --single-branch`) | Blazing fast clone, minimal data transfer.                          |
| Needing to debug when a bug was introduced    | `git bisect` (Reflog helps find start/end)   | Efficient binary search through history.                            |
| Integrating a *private* feature branch        | **Rebasing across branches** (`git rebase`)  | Creates linear, clean history on `main` (vs. merge commit noise).   |
| Following a colleague's WIP branch for review | **Worktree** (or standard checkout)          | Isolate review without disrupting main work (Worktree is cleaner).  |

**Remember the Golden Rules:**
1.  **Never Rewrite Public History:** Only rebase *your own* local commits before they're shared (`push`'ed).
2.  **Reflog is Local Only:** It won't save you from mistakes others made on the remote.
3.  **Shallow Clones are for Limited Use Cases:** Primarily CI/CD, not development.
4.  **Sparse Checkout = Working Directory Only:** `.git` still has full history.
5.  **Worktrees = Multiple Working Copies:** Share one `.git`, avoid branch conflicts.
