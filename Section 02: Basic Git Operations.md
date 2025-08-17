 ### **I. Tracking Files and Making Commits: The Heart of Git**

**Core Principle:** Git tracks *changes*, not files. The **Staging Area** (Index) is Git's "shopping cart" – you select *exactly* which changes to include in your next snapshot (commit).

#### **1. `git add` (Staging Changes)**
*   **What it does:** Moves changes from your **Working Directory** (where you edit files) into the **Staging Area** (a preview of your next commit).
*   **Why it exists:** Gives you **fine-grained control**. You can fix multiple bugs in one file but only commit one fix now.
*   **Key Syntax:**
    *   `git add <file>`: Stage specific file(s).
    *   `git add .` or `git add -A`: Stage **ALL** *new*, *modified*, and *deleted* files in the current directory/subdirectories. **(Use cautiously!)**
    *   `git add -p` or `git add --patch`: **Interactive staging**. Git shows *each change hunk* and asks: Stage this hunk? (y/n/q/a/d/...). *Essential for clean commits.*
*   **Critical Details:**
    *   Staging **only affects the *next* commit**. Subsequent changes to the file *won't* be in that commit unless re-staged.
    *   `git add` **does NOT** track *unmodified* files. Only changes.
    *   **Staging a new file:** Adds the *entire file* (since there's no previous version to diff against).
    *   **Staging a modified file:** Adds *only the specific changes you stage* (via `git add` or `-p`).
    *   **Staging a deleted file:** `git add <file>` after `rm <file>` stages the deletion. `git rm <file>` does delete + stage in one step.
*   **Example Workflow:**
    ```bash
    echo "New feature" >> feature.txt  # Modify file
    echo "Fix bug" >> bugfix.txt      # Modify another file
    git add -p                       # Review changes interactively:
                                     #   - Stage ONLY the "Fix bug" hunk from bugfix.txt
                                     #   - Skip the "New feature" hunk in feature.txt for now
    git commit -m "Fix critical bug" # Commits ONLY the staged bugfix
    git add feature.txt              # Now stage the new feature
    git commit -m "Add cool feature" # Commits ONLY the feature
    ```

#### **2. `git commit` (Creating a Permanent Snapshot)**
*   **What it does:** Takes **everything in the Staging Area** and creates a new, permanent **commit** (snapshot) in your **Local Repository**. This is your *only* chance to save work locally before pushing to a remote.
*   **Why it exists:** Creates an immutable, timestamped, versioned record of your project state at a specific point.
*   **Key Syntax:**
    *   `git commit -m "Meaningful message"`: Commit staged changes with a message *on the command line*.
    *   `git commit`: Opens your default text editor (e.g., Vim) to write a **detailed commit message** (highly recommended for complex changes).
*   **Writing MEANINGFUL Commit Messages (Non-Negotiable Best Practice):**
    *   **Subject Line (50 chars max):** `Capitalize, Imperative Mood, No period`. *What* changed? (e.g., `Fix login validation error`)
    *   **Blank Line:** Mandatory separator.
    *   **Body (Wrap at 72 chars):** *Why* was this change made? *How* does it address the problem? Reference issues (e.g., `Closes #123`). Explain the *consequence* if possible. *Crucial for future you and your team.*
    *   **Footer (Optional):** Breaking changes (`BREAKING CHANGE:`), issue references (`Closes #456`).
    *   **BAD:** `fixed stuff`, `update files`, `commit 1`
    *   **GOOD:**
        ```
        Refactor user authentication module

        - Extracted password validation logic into separate service
        - Simplified JWT token generation flow
        - Added unit tests covering edge cases
        - Fixes intermittent session timeout bug (Issue #789)

        Improves code maintainability and resolves critical user pain point.
        ```
*   **Critical Details:**
    *   **Only staged changes are committed.** Unstaged changes remain in the Working Directory.
    *   Each commit has a **unique SHA-1 hash** (e.g., `a1b2c3d...`), author, timestamp, and parent commit(s).
    *   Commits are **local** until you `git push` to a remote.

#### **3. `git status` (Your Project's Health Check)**
*   **What it does:** Shows the state of your **Working Directory** and **Staging Area** *relative to your last commit*.
*   **Why it exists:** The single most important command for understanding *where your changes are*.
*   **Output Breakdown:**
    *   **Branch Info:** `On branch main`
    *   **Staging Area Status:** `Changes to be committed:` (Green - staged for next commit)
    *   **Working Directory Status:** `Changes not staged for commit:` (Red - modified but *not* staged) & `Untracked files:` (New files not yet tracked)
    *   **Upstream Status:** `Your branch is ahead of 'origin/main' by 1 commit.` (After local commits, before push)
*   **Critical Details:**
    *   **ALWAYS run `git status` before and after staging/committing.** It's your safety net.
    *   Shows which files are modified, staged, untracked, or conflicted.
    *   Tells you if your branch is ahead/behind the remote.

#### **4. `git diff` (Seeing the Differences)**
*   **What it does:** Shows **line-by-line differences** between states in your repository.
*   **Why it exists:** Precisely see *what changed* before staging or committing.
*   **Key Variations:**
    *   `git diff`: **Working Directory vs. Staging Area.** Shows *unstaged* changes (what you'd lose if you discarded changes).
    *   `git diff --staged` or `git diff --cached`: **Staging Area vs. Last Commit.** Shows *staged* changes (what *will* be in the next commit).
    *   `git diff <commit1> <commit2>`: Differences between two specific commits.
    *   `git diff HEAD`: Working Directory vs. *Latest Commit* (shows all uncommitted changes, staged & unstaged).
*   **Critical Details:**
    *   `-` (minus) = Line removed, `+` (plus) = Line added.
    *   **Essential for:** Verifying you staged the *right* changes (`git diff --staged`), reviewing unstaged work (`git diff`), understanding someone else's commit.
    *   **Pro Tip:** Use `git diff --word-diff` for changes within lines (better for prose/docs).

---

### **II. Viewing Commit History: Understanding Your Project's Timeline**

#### **1. `git log` (The Commit Ledger)**
*   **What it does:** Shows the **history of commits** on the *current branch*, starting with the most recent.
*   **Why it exists:** Audit trail, find when a feature/bug was introduced, understand project evolution.
*   **Basic Output (Each commit shows):**
    *   `commit <SHA>`: Unique identifier (e.g., `a1b2c3d4e5f...`)
    *   `Author:` Name & Email
    *   `Date:` Timestamp
    *   Commit message (Subject line only by default)
*   **Powerful Formatting Options:**
    *   `git log --oneline`: **SUPER USEFUL.** `SHA (7 chars) Subject`. Compact history view. (`a1b2c3d Fix login bug`)
    *   `git log --graph`: Shows **branching/merging history** as an ASCII graph. Essential for visualizing complex history.
    *   `git log --all`: Shows commits on **ALL branches** (not just the current one).
    *   `git log --decorate`: Shows **branch and tag names** pointing to commits.
    *   `git log -p`: Shows the **full diff** (patch) for each commit. (`-p` = patch)
    *   `git log -<n>`: Show only the last `n` commits (e.g., `git log -5`).
    *   `git log --author="John"`: Filter commits by author.
    *   `git log --grep="bug"`: Filter commits by message content.
    *   `git log --since="2 weeks ago" --until="1 week ago"`: Time range filtering.
    *   **Combined Power Example:** `git log --oneline --graph --decorate --all` - The ultimate history view.
*   **Critical Details:**
    *   `git log` **only shows commits reachable from the current HEAD** by default. `--all` is crucial for seeing other branches.
    *   Use `q` to quit the pager (like `less`).
    *   **Pro Tip:** Add `alias gl="git log --oneline --graph --decorate --all"` to your `.gitconfig`!

#### **2. `git show` (Inspecting a Single Commit)**
*   **What it does:** Shows **detailed information about a specific commit**, including the full diff.
*   **Why it exists:** Deep dive into *one* commit's changes and metadata.
*   **Key Syntax:**
    *   `git show`: Shows the **most recent commit** (HEAD) and its diff.
    *   `git show <commit>`: Shows details for a specific commit (use SHA, branch name, tag, or relative ref like `HEAD~1`).
    *   `git show <commit>:<file>`: Shows the *contents* of a specific file *as it existed in that commit*.
*   **Output Includes:**
    *   Full commit metadata (SHA, Author, Date, Message)
    *   **Full diff** of all changes made in that commit (like `git diff <commit>^ <commit>`).
*   **Critical Details:**
    *   **The go-to command** for understanding *exactly* what changed in a specific commit.
    *   More focused than `git log -p` (which shows many commits).
    *   Use `git show --name-only` to just see *which files* changed.

---

### **III. Undoing Changes: Git's Safety Net (Use Wisely!)**

**Core Principle:** Git makes it *easy* to undo, but **hard deletes are possible** (especially with `--hard`). **ALWAYS know what state you're undoing FROM and TO.**

#### **1. Undoing in the Working Directory: `git checkout -- <file>`**
*   **What it does:** **Discards changes** in your **Working Directory** for `<file>`, reverting it to the state in the **Staging Area** (or last commit if unstaged).
*   **Why it exists:** Quick "oops" for unwanted edits. **DANGEROUS - irreversible!**
*   **Key Syntax:** `git checkout -- <file>` (The `--` is crucial to separate file from branch names)
*   **Critical Details:**
    *   **ONLY affects the Working Directory.** Staging Area and Repository are untouched.
    *   **DESTROYS uncommitted work** in the specified file(s). **No undo!**
    *   **Use Case:** You made experimental changes to `app.js` and want to nuke them *without* affecting other files/staged changes.
    *   **⚠️ WARNING:** `git checkout <file>` (without `--`) tries to switch branches! **ALWAYS use `--` when undoing file changes.**
    *   **Safer Alternative:** `git restore <file>` (Newer command, same effect, clearer intent).

#### **2. Unstaging Files: `git reset HEAD <file>`**
*   **What it does:** **Removes `<file>` from the Staging Area**, moving it back to "Changes not staged for commit" (Working Directory state). **Does NOT touch your Working Directory files!**
*   **Why it exists:** You accidentally staged the wrong file/change and want to fix your staging before committing.
*   **Key Syntax:** `git reset HEAD <file>` (`HEAD` refers to the last commit)
*   **Critical Details:**
    *   **ONLY affects the Staging Area.** Your actual file changes (Working Directory) remain intact.
    *   **Perfect for:** "I `git add .` but didn't mean to stage `config.local` - unstaged it!"
    *   **After running:** `git status` will show the file under "Changes not staged for commit".
    *   **Safer Alternative:** `git restore --staged <file>` (Newer command, same effect).

#### **3. Amending the Last Commit: `git commit --amend`**
*   **What it does:** **REPLACES the *most recent* commit** with a new commit that includes:
    1.  Any *newly staged changes* (from `git add` after the last commit).
    2.  A potentially *new commit message*.
*   **Why it exists:** Fix small mistakes in the *very last* commit *before* pushing it publicly (typos in message, forgot one file).
*   **Key Syntax:**
    *   `git add <forgotten-file>` (Stage the correction)
    *   `git commit --amend` (Opens editor to fix message *and* incorporates staged changes)
    *   `git commit --amend -m "New message"` (Fix message *only* on command line)
*   **Critical Details:**
    *   **ONLY for the *last* commit on your *current branch*.** Never amend commits already pushed to a shared remote!
    *   **Rewrites history.** The old commit is gone; a new commit with a *different SHA* is created.
    *   **If you've already pushed:** Amending requires `git push --force` (or safer `--force-with-lease`) which **rewrites public history** - **BAD for shared branches!** Only safe for private, unpushed commits.
    *   **Use Case:** You committed "Fix login" but forgot to stage the test file. Stage the test, `commit --amend`, done.

#### **4. Resets: `git reset [--soft|--mixed|--hard] <commit>` (The Big Hammer)**
*   **What it does:** Moves the **`HEAD` pointer** (and optionally the **Staging Area** and **Working Directory**) to a specified `<commit>`. **Powerful but dangerous.**
*   **Why it exists:** To move your branch reference and potentially undo commits/changes at different levels. **Choose your level of destruction carefully!**
*   **The Three Modes (Defined by what they reset):**
    | Mode      | HEAD Pointer | Staging Area | Working Directory | Effect                                                                 | When to Use                                                                 |
    | :-------- | :----------- | :----------- | :---------------- | :--------------------------------------------------------------------- | :-------------------------------------------------------------------------- |
    | **--soft** | **MOVED**    | **UNCHANGED** | **UNCHANGED**     | Last commit(s) undone, changes **STAY STAGED**.                        | You committed to wrong branch? Move commit to new branch (`git branch temp` then `reset --soft HEAD~1`). |
    | **--mixed** (DEFAULT) | **MOVED**    | **RESET**    | **UNCHANGED**     | Last commit(s) undone, changes moved to **WORKING DIRECTORY (unstaged)**. | **MOST COMMON.** Undo last commit but keep your changes to fix/re-stage (`git reset HEAD~1`). |
    | **--hard** | **MOVED**    | **RESET**    | **RESET**         | **DESTROYS** commits AND **DELETES** uncommitted changes in WD & Staging. | **EXTREME CAUTION.** Revert to a known good state, discarding *all* local work since `<commit>`. |

*   **Key Syntax:**
    *   `git reset <commit>`: Default is `--mixed`. (e.g., `git reset HEAD~1` = undo last commit, keep changes unstaged)
    *   `git reset --soft <commit>`
    *   `git reset --hard <commit>` **(⚠️ DANGEROUS - IRREVERSIBLE DATA LOSS!)**
*   **Critical Details:**
    *   `<commit>` can be a SHA, branch name, tag, or relative ref (`HEAD~1`, `HEAD~2`, `main~3`).
    *   **`--hard` is DESTRUCTIVE:** Any uncommitted changes in WD *and* staged changes are **PERMANENTLY LOST**. Double-check `git status` first!
    *   **Resetting Public History:** Like `commit --amend`, resetting commits that have been pushed requires force push and **breaks history for collaborators. AVOID on shared branches.**
    *   **Use Cases:**
        *   `--mixed` (Default): "I committed too early, need to add one more file" -> `git reset HEAD~1` (unstages the commit, keeps changes), `git add correct-file`, `git commit`.
        *   `--soft`: "I committed on `main` but meant to commit on `feature`" -> `git branch feature` (saves current state), `git reset --soft HEAD~1` (moves main back, keeps changes staged), `git checkout feature` (changes are still staged!).
        *   `--hard`: "My local branch is messed up, I want to match exactly what's on `origin/main`" -> `git fetch origin`, `git reset --hard origin/main`. **(Only safe if you KNOW you want to discard ALL local changes!)**

---

### **IV. Ignoring Files: Keeping Git Clean**

#### **1. The `.gitignore` File**
*   **What it is:** A **text file** in your repository root (or subdirectories) listing **patterns** for files/directories Git should **NEVER track**.
*   **Why it exists:** Exclude generated files (binaries, logs), secrets, IDE configs, OS artifacts, and other noise from version control.
*   **How it Works:**
    *   Git checks `.gitignore` **ONLY for *untracked* files.** If a file is *already tracked*, `.gitignore` has **NO EFFECT** on it. (You must `git rm --cached <file>` to stop tracking it).
    *   Patterns are matched against file paths relative to the `.gitignore` file's location.
*   **Pattern Syntax (Examples):**
    *   `*.log`: Ignore **all** files ending with `.log` (e.g., `app.log`, `error.log`).
    *   `logs/`: Ignore **entire directory** named `logs` (and its contents).
    *   `temp*.txt`: Ignore files starting with `temp` and ending with `.txt` (e.g., `temp1.txt`).
    *   `!important.log`: **Un-ignore** `important.log` (if a previous pattern would have ignored it). `!` negates.
    *   `/build/`: Ignore `build/` **only in the root directory** (not in subdirs). Leading `/` anchors to root.
    *   `**/temp`: Ignore `temp` directory **anywhere** in the repo (recursive).
    *   `config.local`: Ignore a specific file.
*   **Critical Details:**
    *   **Place `.gitignore` in your repo root.** Git automatically finds it.
    *   **Commit `.gitignore`!** It's part of your project configuration.
    *   **Common Patterns to Ignore:**
        *   Build outputs: `bin/`, `obj/`, `dist/`, `*.exe`, `*.dll`, `*.class`, `*.o`, `*.a`
        *   Logs: `*.log`, `logs/`
        *   Dependencies: `node_modules/`, `vendor/`, `packages/`
        *   IDE/Editor: `.idea/`, `.vscode/`, `*.suo`, `*.ntvs*`, `*.swp`, `*.swn`
        *   OS: `.DS_Store`, `Thumbs.db`, `Desktop.ini`
        *   Secrets: `*.env`, `secrets.json`, `config.local`
    *   **If a file is already tracked:** `.gitignore` won't hide it. Use `git rm --cached <file>` to stop tracking it (deletes it from repo but leaves it in WD), then add the pattern to `.gitignore`.

#### **2. Global `.gitignore`**
*   **What it is:** A `.gitignore` file **applied to *all* repositories on your machine**.
*   **Why it exists:** Ignore files specific to *your* OS or tools (e.g., `.DS_Store`, `*.swo`) without polluting every project's `.gitignore`.
*   **How to Set Up:**
    1.  Create the file: `touch ~/.gitignore_global`
    2.  Configure Git to use it: `git config --global core.excludesfile ~/.gitignore_global`
    3.  Add your global patterns (e.g., `.DS_Store`, `*.swp`, `Thumbs.db`)
*   **Critical Details:**
    *   Affects **every Git repo** on your system.
    *   **Does NOT get committed** to any repository.
    *   **Use for:** Truly personal, machine-specific ignores. Project-specific ignores belong in the repo's `.gitignore`.

---

### **V. Working with Remotes: Collaboration is Key**

**Core Principle:** A **remote** is a **URL pointing to another copy** of your repository (usually on GitHub, GitLab, Bitbucket, or a server). `origin` is the conventional name for the *main* remote you cloned from.

#### **1. Adding Remote Repositories: `git remote add`**
*   **What it does:** Creates a **shortname** (like `origin`) that points to a remote repository URL.
*   **Why it exists:** Tell your local repo where the "central" or other team repos are located.
*   **Key Syntax:** `git remote add <shortname> <url>`
    *   `git remote add origin https://github.com/user/repo.git` (Most common - sets up the "main" remote)
    *   `git remote add upstream https://github.com/original/repo.git` (Common for forks - points to original project)
*   **Critical Details:**
    *   `origin` is a **convention**, not a requirement. You can name remotes anything (`upstream`, `staging`, `prod`).
    *   The URL can be HTTPS (`https://...`) or SSH (`git@github.com:user/repo.git`). SSH is preferred for automation (uses keys).
    *   Adding a remote **does NOT fetch data** yet. It just sets up the pointer.

#### **2. Viewing Remotes: `git remote -v`**
*   **What it does:** Lists **all configured remotes** and their **fetch/push URLs**.
*   **Why it exists:** Verify your remotes are set up correctly.
*   **Key Syntax:**
    *   `git remote`: Lists remote *names* only (e.g., `origin`, `upstream`).
    *   `git remote -v`: **MOST USEFUL.** Lists names **AND** their fetch/push URLs (e.g., `origin  https://... (fetch)`, `origin  https://... (push)`).
*   **Critical Details:**
    *   Essential for troubleshooting push/pull issues ("Am I pushing to the right place?").
    *   Shows if fetch and push URLs differ (sometimes useful, but usually the same).

#### **3. Fetching: `git fetch`**
*   **What it does:** **Downloads *all* new commits, branches, and tags** from the specified remote **into your *local* repository** (into `refs/remotes/<remote>/` namespace, e.g., `origin/main`). **Does NOT merge or touch your Working Directory!**
*   **Why it exists:** See what others have done *without* altering your current work. Safe inspection.
*   **Key Syntax:**
    *   `git fetch`: Fetches from **default remote** (`origin`).
    *   `git fetch origin`: Explicitly fetches from `origin`.
    *   `git fetch --all`: Fetches from **all** configured remotes.
*   **Critical Details:**
    *   **SAFEST remote command.** Only updates your *local copy* of the remote's state (`origin/main`), **not** your `main` branch or files.
    *   After `git fetch`, you can:
        *   `git log origin/main`: See new commits others pushed.
        *   `git diff main..origin/main`: See differences between your branch and remote.
        *   `git merge origin/main` or `git rebase origin/main`: Integrate the changes (see Pull below).
    *   **Crucial for:** Reviewing changes before integrating, checking if you're behind.

#### **4. Pulling: `git pull`**
*   **What it does:** **`git fetch` + `git merge`** (by default). Fetches changes from the remote **and immediately merges them** into your **current branch**.
*   **Why it exists:** Convenience command to update your current branch with the latest from its tracking remote branch.
*   **Key Syntax:**
    *   `git pull`: Pulls from the **upstream branch** configured for your *current* branch (e.g., `main` pulls from `origin/main`).
    *   `git pull origin main`: Explicitly pulls `main` from `origin` into your current branch.
*   **Critical Details:**
    *   **Equivalent to:** `git fetch` followed by `git merge origin/<current-branch>`.
    *   **Can cause merge conflicts** if your local changes and remote changes overlap.
    *   **Default Behavior:** Uses `merge`. You can configure it to use rebase instead (`git config --global pull.rebase true`), which is often cleaner: `git pull` = `git fetch` + `git rebase origin/<current-branch>`.
    *   **When to Use:** When you want to quickly update your current branch with the latest shared changes *and* you're ready to resolve potential conflicts.
    *   **When NOT to Use:** If you want to *review* changes first (use `git fetch` then inspect). If you have uncommitted work (commit/stash first!).

#### **5. Pushing: `git push`**
*   **What it does:** **Uploads your local commits** to a specified remote repository and **updates the remote branch**.
*   **Why it exists:** Share your work with others and back up your commits.
*   **Key Syntax:**
    *   `git push`: Pushes **current branch** to its **upstream branch** (e.g., `main` -> `origin/main`). Requires upstream to be set (usually done on first push).
    *   `git push -u origin main`: **First push to a new branch.** `-u` sets upstream tracking (`main` -> `origin/main`), so future `git push`/`git pull` work without arguments.
    *   `git push origin main`: Pushes local `main` to remote `origin`'s `main`.
    *   `git push --force` or `git push --force-with-lease`: **DANGEROUS.** Overwrites remote branch history. **Only use if you *know* you won't break others' work** (e.g., fixing your *own* private branch after `commit --amend`).
*   **Critical Details:**
    *   **Only pushes commits *reachable* from your current branch.**
    *   **Fails if the remote has new commits you don't have** (you need to `git pull` first to integrate). This is the "non-fast-forward" error.
    *   **`-u` (or `--set-upstream`)** is vital for new branches to establish the tracking relationship.
    *   **`--force-with-lease` is SAFER than `--force`:** It only forces if the remote branch hasn't changed since *you last fetched*. Prevents accidentally overwriting someone else's new work.
    *   **Never `--force` a shared branch** (like `main` or `develop`) that others are using. Coordinate carefully!

---

### **Key Mental Models for Success**

1.  **The Three Rooms:**
    *   **Working Directory:** Your files on disk (editing space).
    *   **Staging Area (Index):** Your "proposed next commit" (shopping cart).
    *   **Repository (Local):** All your committed snapshots (permanent storage).
    *   **Remote Repository:** Another copy (usually on a server) of the Repository.

2.  **`git status` is Your Compass:** Always know which room your changes are in.

3.  **Commits are Immutable Snapshots:** Once committed (and pushed), treat them as permanent history. Undoing *after* push is complex.

4.  **Remotes are Separate Copies:** `fetch`/`pull`/`push` move commits *between* repository copies. They don't automatically sync.

5.  **Undoing is Contextual:** Know *where* your changes are (WD, Staging, Repo) and *how far back* you need to go before choosing a command (`checkout`, `reset`, `amend`).
