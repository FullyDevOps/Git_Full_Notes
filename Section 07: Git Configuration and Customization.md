### 📁 **1. Git Configuration: The 3-Tier Hierarchy**
Git stores settings in **three distinct levels**, with **lower levels overriding higher ones**. Understanding this hierarchy is crucial.

#### **A. System-Level Configuration (`--system`)**
*   **Purpose:** Applies to **ALL users** on the machine. For system-wide defaults.
*   **Location:**
    *   Linux/macOS: `/etc/gitconfig`
    *   Windows: `C:\Program Files\Git\etc\gitconfig` (or where Git is installed)
*   **How to Set:**
    ```bash
    git config --system user.name "System Admin"
    git config --system core.editor "vim"
    ```
*   **Key Nuances:**
    *   Requires **administrative/root privileges** to modify.
    *   Rarely used for personal settings. Primarily for org-wide defaults (e.g., enforcing commit message formats).
    *   **Override Priority:** Lowest (easiest to override).

#### **B. Global/User-Level Configuration (`--global`)**
*   **Purpose:** Applies to **ALL repositories** for the **current user**. Your personal defaults.
*   **Location:**
    *   Linux/macOS: `~/.gitconfig` or `~/.config/git/config`
    *   Windows: `C:\Users\<YourUser>\.gitconfig`
*   **How to Set (MOST COMMON):**
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "you@example.com"
    git config --global core.editor "code --wait"  # VS Code
    ```
*   **Key Nuances:**
    *   **This is where 90% of personal customization happens.**
    *   Settings apply to **every repo** you work in *unless* overridden locally.
    *   **View All Global Settings:**
        ```bash
        git config --global --list
        git config --global -l  # Shorter
        ```

#### **C. Local/Repository-Level Configuration (`--local` - DEFAULT)**
*   **Purpose:** Applies **ONLY** to the **specific repository** you're currently in. Overrides system/global.
*   **Location:** `.git/config` **INSIDE** your repo directory (e.g., `/your/repo/.git/config`). *Not visible in your project files!*
*   **How to Set:**
    ```bash
    # Inside a repo directory
    git config user.email "work@company.com"  # Overrides global email for THIS repo
    git config core.autocrlf input           # Repo-specific line ending handling
    ```
*   **Key Nuances:**
    *   **Default level** if you omit `--system`, `--global`, or `--local`.
    *   **Critical for:** Project-specific settings (e.g., different email for work vs. personal repos, repo-specific hooks).
    *   **View Local Settings:**
        ```bash
        git config --local --list
        ```

#### **D. Viewing & Editing Configuration**
*   **View ALL Config (All Levels):**
    ```bash
    git config --list  # Shows effective config (local > global > system)
    git config -l      # Shorter
    ```
*   **View Source of a Specific Setting:**
    ```bash
    git config --show-origin user.name
    ```
*   **Edit Config Directly:**
    ```bash
    git config --global --edit  # Opens ~/.gitconfig in $EDITOR
    git config --local --edit   # Opens .git/config in $EDITOR
    ```

---

### ⚡ **2. Aliases: Turbocharge Your Git CLI**
Create shortcuts for complex or frequently used commands.

#### **A. Creating Aliases**
*   **Global Alias (Recommended - persists across repos):**
    ```bash
    git config --global alias.co checkout
    git config --global alias.br branch
    git config --global alias.ci commit
    git config --global alias.st status
    git config --global alias.last "log -1 HEAD"  # Show last commit
    ```
*   **Local Alias (Repo-specific):**
    ```bash
    git config alias.graph "log --graph --oneline --decorate --all"
    ```

#### **B. Advanced Aliases (Shell Commands)**
Use `!` to execute shell commands. **Crucial for complex workflows.**
```bash
# Global alias for a fancy log
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# Global alias to prune stale remote-tracking branches
git config --global alias.prune-remote "!git fetch --prune && git remote prune origin"

# Global alias to amend last commit *without* editing message
git config --global alias.amend "commit --amend --no-edit"
```
*   **⚠️ Critical Note:** Aliases using `!` are **shell commands**. Ensure your shell (bash/zsh) understands them. Windows CMD/PowerShell might need adjustments.

#### **C. Viewing Aliases**
```bash
git config --get-regexp alias  # Lists all aliases
git config --global --get-regexp alias  # Global only
```

#### **D. Executing Aliases**
*   `git co` = `git checkout`
*   `git lg` = Your fancy log command
*   `git prune-remote` = Runs the fetch/prune command

---

### 🎨 **3. Color Settings: Make Git Output Readable**
Git uses color by default in terminals, but you can fine-tune it.

#### **A. Enable/Disable Color**
```bash
git config --global color.ui auto  # Default - color when output is a terminal
git config --global color.ui true  # Always use color
git config --global color.ui false # Never use color
```

#### **B. Fine-Grained Color Control**
*   **Status Command:**
    ```bash
    git config --global color.status.added green
    git config --global color.status.changed yellow
    git config --global color.status.untracked cyan
    ```
*   **Branch Command:**
    ```bash
    git config --global color.branch.current green
    git config --global color.branch.local yellow
    git config --global color.branch.remote blue
    ```
*   **Diff Command (Most Useful):**
    ```bash
    git config --global color.diff.meta "yellow bold"  # Commit headers
    git config --global color.diff.frag "magenta bold" # +/- line numbers
    git config --global color.diff.old "red"          # Removed lines
    git config --global color.diff.new "green"        # Added lines
    git config --global color.diff.commit "yellow"    # commit hash line
    ```

#### **C. Viewing Color Settings**
```bash
git config --global --get-regexp color
```

---

### 🔌 **4. Diff & Merge Tools: Resolve Conflicts Like a Pro**
Git uses built-in tools by default, but external GUI tools are far superior for complex conflicts.

#### **A. Configuring an External Diff Tool**
*   **Example: Beyond Compare (Cross-Platform)**
    ```bash
    git config --global diff.tool bc
    git config --global difftool.bc.path "/path/to/bcomp"  # Linux/macOS
    git config --global difftool.bc.path "C:/Program Files/Beyond Compare 4/BCompare.exe"  # Windows
    git config --global difftool.bc.trustExitCode true
    ```
*   **Usage:**
    ```bash
    git difftool file.txt  # Compares working copy vs. index
    git difftool HEAD~1 HEAD file.txt  # Compares two commits
    ```

#### **B. Configuring an External Merge Tool (For Conflicts)**
*   **Example: P4Merge (Free & Excellent)**
    ```bash
    # Tell Git which tool to use
    git config --global merge.tool p4merge

    # Specify the path to the executable
    git config --global mergetool.p4merge.path "/path/to/p4merge"  # Linux/macOS
    git config --global mergetool.p4merge.path "C:/Program Files/Perforce/p4merge.exe"  # Windows

    # Tell Git how to invoke it (CRITICAL - paths vary by OS/tool)
    # Linux/macOS (P4Merge):
    git config --global mergetool.p4merge.cmd 'p4merge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'

    # Windows (P4Merge - Note: Double quotes & escaping):
    git config --global mergetool.p4merge.cmd '"C:/Program Files/Perforce/p4merge.exe" "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'

    # Tell Git to trust the tool's exit code (0 = success, non-0 = failure)
    git config --global mergetool.p4merge.trustExitCode false
    ```
*   **Usage During Conflict:**
    1.  Git detects a conflict during `merge`/`rebase`/`cherry-pick`.
    2.  Run: `git mergetool`
    3.  P4Merge (or your tool) launches for **each conflicted file**.
    4.  Resolve conflicts visually within the tool.
    5.  Save the resolved file in the tool -> Git marks it as resolved.
    6.  Run `git commit` to complete the merge.

#### **C. Popular Tools & Paths**
| Tool          | Linux/macOS Path                     | Windows Path                                      |
|---------------|--------------------------------------|---------------------------------------------------|
| **Beyond Compare** | `/usr/bin/bcomp`                   | `C:/Program Files/Beyond Compare 4/BCompare.exe` |
| **Meld**        | `/usr/bin/meld`                    | `C:/Program Files (x86)/Meld/meld.exe`           |
| **P4Merge**     | `/usr/local/bin/p4merge`           | `C:/Program Files/Perforce/p4merge.exe`          |
| **VS Code**     | `code --wait`                      | `C:/Users/<User>/AppData/Local/Programs/Microsoft VS Code/Code.exe` |
| **KDiff3**      | `/usr/bin/kdiff3`                  | `C:/Program Files/KDiff3/kdiff3.exe`             |

#### **D. Critical Notes for Diff/Merge Tools**
1.  **Install First:** Git **only configures** the tool; you **MUST install it separately**.
2.  **Path Accuracy:** Paths **MUST be exact**. Use quotes on Windows. Test the path in your terminal.
3.  **Command Syntax:** The `cmd` setting (`$BASE`, `$LOCAL`, etc.) is **tool-specific**. Consult your tool's docs. Git passes these placeholders:
    *   `$LOCAL`: Your side (current branch)
    *   `$REMOTE`: Incoming changes (branch being merged)
    *   `$BASE`: Common ancestor version
    *   `$MERGED`: File where Git expects the resolution.
4.  **Trust Exit Code:** Set `trustExitCode` based on the tool. `false` is safer (Git checks if `$MERGED` changed).
5.  **VS Code Tip:** Use `code --wait` for diff/merge so Git waits for resolution.

---

### 🪝 **5. Custom Git Hooks: Automate Your Workflow**
Hooks are **scripts** Git executes automatically at specific points. They live in `.git/hooks/` **inside your repo**.

#### **A. Client-Side Hooks (Run on YOUR machine)**
*   **Location:** `.git/hooks/` (in **your** local repo)
*   **Activation:** Copy template file (e.g., `pre-commit.sample`) to `pre-commit` (remove `.sample`) and make it **executable** (`chmod +x pre-commit`).
*   **Key Hooks:**
    *   **`pre-commit`:** Runs **BEFORE** commit is recorded. **Ideal for:**
        *   Linting code (`eslint`, `flake8`)
        *   Running tests (`npm test`, `pytest`)
        *   Checking for secrets (`trufflehog`, `git-secrets`)
        *   **Example (Bash - Block commits with "WIP"):**
            ```bash
            #!/bin/sh
            if grep -q "WIP" $(git diff --cached --name-only); then
              echo "Commit rejected: Found 'WIP' in files. Finish your work first!"
              exit 1
            fi
            exit 0
            ```
    *   **`prepare-commit-msg`:** Modify commit message *before* editor opens (e.g., add ticket #).
    *   **`commit-msg`:** Validate commit message format (e.g., enforce Conventional Commits).
    *   **`pre-push`:** Runs **AFTER** `pre-commit` but **BEFORE** `git push` contacts remote. **Ideal for:**
        *   Running comprehensive tests (`npm run test:ci`)
        *   Checking for large files (`git diff --cached --name-only | xargs ls -lh`)
        *   **Example (Python - Check file size < 10MB):**
            ```python
            #!/usr/bin/env python3
            import sys
            import subprocess

            # Get list of files being pushed
            changed_files = subprocess.check_output(['git', 'diff', '--cached', '--name-only']).decode().splitlines()

            for file in changed_files:
                # Get file size in bytes
                size = int(subprocess.check_output(['wc', '-c', file]).split()[0])
                if size > 10_000_000:  # 10MB
                    print(f"ERROR: {file} is {size} bytes (>10MB). Push rejected.")
                    sys.exit(1)
            sys.exit(0)
            ```
    *   **`post-commit`:** Runs **AFTER** commit is recorded. Good for notifications (e.g., `echo "Committed $(git rev-parse HEAD)"`).
    *   **`post-merge`:** Runs **AFTER** a successful `git merge` or `git pull`. Useful for:
        *   Installing dependencies (`npm install`, `bundle install`)
        *   Clearing caches
        *   **Example (Bash - Install deps after merge):**
            ```bash
            #!/bin/sh
            if [ -f package.json ]; then
              npm install --silent
            fi
            ```

#### **B. Server-Side Hooks (Run on REMOTE repo - e.g., GitHub/GitLab)**
*   **Location:** `hooks/` directory **INSIDE** the bare repo on the server (e.g., `/var/repo/project.git/hooks/`).
*   **Activation:** Same as client hooks (rename `.sample`, `chmod +x`). **Requires server admin access.**
*   **Key Hooks:**
    *   **`pre-receive`:** Runs **ONCE** **BEFORE** any refs are updated. **MOST POWERFUL.**
        *   **Rejects the ENTIRE push** if it exits non-zero.
        *   **Ideal for:**
            *   Enforcing branch naming conventions
            *   Blocking commits to `main`/`master`
            *   Validating commit messages globally
            *   Checking for secrets in *all* pushed commits
        *   **Input:** Reads `stdin` in format: `<old-value> <new-value> <ref-name>`
    *   **`update`:** Runs **ONCE PER REF** being updated (e.g., once per branch/tag pushed).
        *   **Rejects ONLY the specific ref** if it exits non-zero.
        *   **Ideal for:** Per-branch rules (e.g., "only QA can push to `release/*`").
        *   **Arguments:** `$1` = ref name, `$2` = old SHA, `$3` = new SHA.
    *   **`post-receive`:** Runs **ONCE** **AFTER** all refs are updated. **MOST COMMON FOR AUTOMATION.**
        *   **Ideal for:**
            *   Triggering deployments (`ssh webserver "git pull"`)
            *   Sending notifications (Slack, email)
            *   Updating issue trackers
        *   **Input:** Same `stdin` format as `pre-receive`.
        *   **Example (Bash - Deploy main branch):**
            ```bash
            #!/bin/sh
            while read oldrev newrev ref
            do
                if [ "$ref" = "refs/heads/main" ]; then
                    echo "Deploying main branch to production..."
                    ssh deploy@prod-server "cd /var/www && git pull"
                fi
            done
            ```

#### **C. Critical Hook Rules**
1.  **File Names:** Must be exact (e.g., `pre-commit`, **NOT** `pre-commit.sh`).
2.  **Permissions:** **MUST be executable** (`chmod +x hook-name`).
3.  **Shebang:** First line **MUST specify interpreter** (`#!/bin/sh`, `#!/usr/bin/env python3`).
4.  **Exit Codes:**
    *   `0` = Success (proceed)
    *   **Non-zero** = Failure (reject operation for client hooks; reject ref/push for server hooks)
5.  **Bare Repos:** Server hooks only work on **bare repositories** (`git init --bare`).
6.  **No Git Commands:** Avoid `git commit` inside hooks (causes recursion). Use `git stash` cautiously.

#### **D. Husky: Git Hooks for Node.js Projects (Simplifies Client Hooks)**
*   **Problem:** Native hooks aren't tracked in Git. Hard to share across team members.
*   **Solution:** Husky manages hooks via `package.json`.
*   **Setup:**
    1.  Install: `npm install husky --save-dev`
    2.  Enable: `npx husky install` (creates `.husky/` dir)
    3.  Add Hook: `npx husky add .husky/pre-commit "npm test"`
*   **How it Works:**
    *   Husky creates a `.husky/pre-commit` script.
    *   This script runs your command (`npm test`).
    *   **Commit is blocked** if the command fails (exit code != 0).
*   **Benefits:**
    *   Hooks are **version controlled** (in `.husky/`).
    *   Easy setup via npm scripts.
    *   Cross-platform (handles Windows paths better).
*   **Common Husky Hooks:**
    ```json
    // package.json
    {
      "scripts": {
        "lint": "eslint .",
        "test": "jest"
      },
      "husky": {
        "hooks": {
          "pre-commit": "npm run lint && npm run test",
          "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
        }
      }
    }
    ```
*   **⚠️ Important:** Husky **only manages client-side hooks**. Server-side hooks still require manual setup on the remote.

---

### 🧩 **6. Resolving Conflicts Visually: Step-by-Step**
When `git merge`/`rebase`/`cherry-pick` fails due to conflicts:

1.  **Identify Conflicts:**
    ```bash
    git status  # Shows "Unmerged paths" and conflicted files
    ```
2.  **Open Merge Tool:**
    ```bash
    git mergetool  # Launches your configured tool (e.g., P4Merge) for each conflicted file
    ```
3.  **Inside the Merge Tool (e.g., P4Merge):**
    *   **Left Pane:** `$LOCAL` (Your changes - current branch)
    *   **Right Pane:** `$REMOTE` (Incoming changes - branch being merged)
    *   **Top Pane:** `$BASE` (Common ancestor - where changes diverged)
    *   **Bottom Pane:** `$MERGED` (Where you resolve the conflict)
    *   **Resolve:** Click arrows (`<<` or `>>`) to choose LOCAL or REMOTE version for each conflict block. Edit manually if needed.
    *   **Save:** Save the resolved file **in the merge tool**. This updates `$MERGED`.
4.  **Mark as Resolved:**
    *   The tool exits -> Git automatically marks the file as resolved.
    *   Verify: `git status` should no longer list the file under "Unmerged paths".
5.  **Complete the Operation:**
    ```bash
    git commit  # Or `git rebase --continue` / `git cherry-pick --continue`
    ```

#### **Key Conflict Resolution Tips**
*   **Understand the Conflict Markers (If editing manually):**
    ```
    <<<<<<< HEAD
    Your changes (current branch)
    =======
    Their changes (incoming branch)
    >>>>>>> branch-name
    ```
    *   `HEAD` = Your side (current state)
    *   `branch-name` = Incoming changes
*   **Use `git checkout --ours` / `--theirs`:** Resolve entire file to one side (DANGEROUS - use carefully!):
    ```bash
    git checkout --ours conflicted_file.txt  # Keep YOUR version
    git checkout --theirs conflicted_file.txt # Keep THEIR version
    ```
*   **Abort:** If stuck, `git merge --abort` / `git rebase --abort` / `git cherry-pick --abort`.

---

### ✅ **Summary Cheat Sheet**

| **Category**         | **Key Command/Concept**                                  | **Purpose**                                                                 |
|----------------------|----------------------------------------------------------|-----------------------------------------------------------------------------|
| **Config Levels**    | `git config --system`                                    | System-wide (all users)                                                     |
|                      | `git config --global`                                    | User-wide (all repos) - **MOST USED**                                       |
|                      | `git config` (no flag)                                   | Repo-specific (current repo)                                                |
| **Aliases**          | `git config --global alias.co checkout`                  | Create shortcut (`git co` = `git checkout`)                                 |
|                      | `git config --global alias.lg "log --graph..."`          | Complex command shortcut                                                    |
| **Color**            | `git config --global color.ui auto`                      | Enable color (default)                                                      |
|                      | `git config --global color.diff.old red`                 | Customize diff colors                                                       |
| **Diff Tool**        | `git config --global diff.tool bc`                       | Set diff tool (e.g., Beyond Compare)                                        |
|                      | `git difftool file.txt`                                  | Run diff tool                                                               |
| **Merge Tool**       | `git config --global merge.tool p4merge`                 | Set merge tool (e.g., P4Merge)                                              |
|                      | `git config --global mergetool.p4merge.cmd '...'`        | Define *how* to launch the tool                                             |
|                      | `git mergetool`                                          | Resolve conflicts visually                                                  |
| **Client Hooks**     | `.git/hooks/pre-commit` (executable)                     | Run before commit (lint, tests)                                             |
|                      | `.git/hooks/pre-push` (executable)                       | Run before push (comprehensive tests)                                       |
|                      | `.git/hooks/post-merge` (executable)                     | Run after merge (install deps)                                              |
| **Server Hooks**     | `repo.git/hooks/pre-receive` (executable)                | **Reject entire push** (enforce rules)                                      |
|                      | `repo.git/hooks/post-receive` (executable)               | **Trigger deployments/notifications**                                       |
| **Husky**            | `npx husky add .husky/pre-commit "npm test"`             | Manage client hooks via npm (Node.js)                                       |
| **Conflict Resolve** | `git mergetool` → Resolve in GUI → `git commit`          | Visual conflict resolution workflow                                         |

---
