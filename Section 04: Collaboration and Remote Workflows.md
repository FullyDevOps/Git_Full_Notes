### **Section 04: Collaboration and Remote Workflows**

#### **1. Working with GitHub, GitLab, Bitbucket**
*(The Foundation of Remote Collaboration)*

*   **What They Are**: Cloud-based platforms hosting Git repositories with **added collaboration features** (PRs/MRs, Issues, CI/CD, Wikis). Git is the underlying version control system; these are *hosts*.
*   **Key Differences**:
    *   **GitHub**: Largest OSS community, strongest PR culture, GitHub Actions (CI/CD), extensive marketplace. *Dominant in OSS*.
    *   **GitLab**: All-in-one DevOps platform (built-in CI/CD, Container Registry, Kubernetes, Monitoring). Stronger focus on *enterprise* and *self-hosting*. Merge Requests are central.
    *   **Bitbucket**: Tight integration with **Jira** and **Confluence** (Atlassian suite). Better for *private repos* in enterprise teams already using Atlassian tools. Supports both Git and Mercurial (though Git is standard now).

*   **Creating Repositories Online**
    *   **Why**: Centralized starting point for collaboration. Avoids manual `git init` + `git remote add` setup.
    *   **How**:
        1.  Log into platform (GitHub/GitLab/Bitbucket).
        2.  Click "New Repository" / "Create repository".
        3.  **Critical Settings**:
            *   **Name**: Short, descriptive (e.g., `project-api`, `frontend-ui`).
            *   **Visibility**: `Public` (anyone can see, limited write access) vs. `Private` (only invited collaborators).
            *   **Initialize**: *Optional but recommended*: Add `.gitignore` (e.g., for Python, Node.js), `LICENSE`, `README.md`. **Crucial**: If you initialize with these, the platform creates the *first commit* (`initial commit`), giving you something to clone. *If you skip this, you MUST `git init` locally first before pushing*.
        4.  Click "Create".
    *   **Post-Creation**: Platform provides **HTTPS** and **SSH** clone URLs (see below). You now have a *bare repository* (no working files) on the server.

*   **Forking Repositories**
    *   **What**: Creating a **personal copy** of *someone else's repository* (usually OSS) under *your own account/namespace*. It's a *server-side copy*, not a local clone.
    *   **Why**:
        *   **OSS Contribution**: You don't have write access to the original ("upstream") repo. Forking gives you a space to make changes.
        *   **Experimentation**: Safely try changes without affecting the original project.
        *   **Starting Point**: Use an existing project as a base for your own (with proper attribution/licensing!).
    *   **How**:
        1.  Navigate to the repo you want to fork (e.g., `https://github.com/original-owner/project`).
        2.  Click the **"Fork"** button (top-right on GitHub/GitLab, top on Bitbucket).
        3.  Select your **account/organization** as the destination.
        4.  Platform creates `your-username/project` (or `your-org/project`).
    *   **Key Nuance**: The fork is *linked* to the upstream repo. Platforms track this relationship, enabling PRs *from* your fork *to* upstream. **Forking does NOT automatically clone to your local machine** – you still need to `git clone` your fork.

*   **Cloning Public and Private Repos**
    *   **What**: Creating a **local copy** of a *remote repository* (on GitHub/GitLab/Bitbucket) onto your development machine. Includes *all* history and branches.
    *   **Why**: To work on the code locally (edit, commit, test).
    *   **How** (`git clone <URL>`):
        *   **Public Repo (Read-Only)**:
            *   `git clone https://github.com/owner/public-repo.git` (HTTPS)
            *   `git clone git@github.com:owner/public-repo.git` (SSH)
            *   *No credentials needed for cloning*, but **pushing requires auth**.
        *   **Private Repo (Read/Write)**:
            *   Requires authentication for **both clone and push**.
            *   HTTPS: Prompts for username/password (or **Personal Access Token - PAT** - *passwords often don't work anymore!*).
            *   SSH: Uses your SSH key (see below).
    *   **Critical Step**: After `git clone`, your local repo has a remote named `origin` pointing to *your fork* (if you cloned your fork) or the *upstream repo* (if you cloned directly). **To contribute to upstream via a fork, you MUST add the upstream repo as a separate remote** (see "Syncing with Upstream").

*   **SSH vs HTTPS Setup**
    *   **The Core Difference**: **How Git authenticates** you to the remote server (GitHub/GitLab/Bitbucket).
    *   **HTTPS**:
        *   **URL Format**: `https://github.com/username/repo.git`
        *   **Authentication**:
            *   Requires **username** + **password** OR (more securely/commonly) **Personal Access Token (PAT)**.
            *   **PAT is MANDATORY** on most platforms for HTTPS Git operations (passwords often rejected). Created in your account settings (e.g., GitHub "Developer Settings > Personal Access Tokens").
            *   *Pros*: Works behind most firewalls/proxies. Easier for beginners initially.
            *   *Cons*: Requires entering PAT frequently (unless cached by `git-credential-manager`). Less secure if PAT is mishandled (stored in plaintext).
        *   **Setup**:
            1.  Generate PAT with `repo` scope (and `admin:org`, `workflow` if needed).
            2.  Use PAT *instead of password* when prompted during `git push`/`git clone`.
            3.  *(Optional)* Configure Git to cache credentials: `git config --global credential.helper cache` (stores in memory for 15 mins) or `store` (plaintext file - *less secure*).
    *   **SSH**:
        *   **URL Format**: `git@github.com:username/repo.git`
        *   **Authentication**:
            *   Uses **public-key cryptography**. You have a *private key* (securely stored on your machine) and a *public key* (added to your GitHub/GitLab/Bitbucket account).
            *   *Pros*: More secure (no passwords/PATs typed). Seamless authentication after setup (`git push` just works). Required for some advanced automation.
            *   *Cons*: Initial setup is trickier. Firewalls might block port 22 (SSH). Requires managing key pairs.
        *   **Setup (Critical Steps)**:
            1.  **Generate Key Pair** (if none exists): `ssh-keygen -t ed25519 -C "your_email@example.com"` (Press Enter for defaults). *Avoid passphrase for simplicity in CI, but use one locally for security*.
            2.  **Start SSH Agent**: `eval "$(ssh-agent -s)"` (Linux/macOS) / `Get-Service -Name ssh-agent | Set-Service -StartupType Automatic` then `Start-Service ssh-agent` (Windows PowerShell).
            3.  **Add Private Key to Agent**: `ssh-add ~/.ssh/id_ed25519` (or your key path).
            4.  **Copy Public Key**: `cat ~/.ssh/id_ed25519.pub` (Linux/macOS) / `Get-Content ~/.ssh/id_ed25519.pub` (Windows). **Copy the ENTIRE string** (starts with `ssh-ed25519 ...`).
            5.  **Add Public Key to Platform**:
                *   GitHub: Settings > SSH and GPG keys > New SSH key
                *   GitLab: Preferences > SSH Keys
                *   Bitbucket: Personal settings > SSH keys
            6.  **Verify**: `ssh -T git@github.com` (GitHub) / `ssh -T git@gitlab.com` (GitLab) / `ssh -T git@bitbucket.org` (Bitbucket). Should get a success message.
        *   **Which to Choose?** **SSH is strongly recommended** for security and convenience once set up. Use HTTPS only if SSH is blocked or for very simple public repo reads.

---

#### **2. Pull Requests (GitHub, Bitbucket) / Merge Requests (GitLab)**
*(The Heart of Collaborative Code Review)*

*   **What They Are**: **Proposals** to integrate changes from one branch (usually a feature/fix branch) *into* another branch (usually `main`/`master` or `develop`). They are **not Git commands** – they are platform features built *on top* of Git.
*   **Why They Exist**:
    *   **Code Review**: Mandatory step for quality, knowledge sharing, catching bugs.
    *   **Discussion**: Contextual conversation about the change *on the code*.
    *   **Automation Trigger**: Kicks off CI/CD pipelines (tests, builds).
    *   **Audit Trail**: Records *who* changed *what*, *why*, and *when* approval happened.
    *   **Gatekeeping**: Enforces branch protection rules before merging.

*   **Creating PRs/MRs**
    *   **Prerequisites**:
        1.  You have changes committed on a **branch** (NOT `main`/`master`!).
        2.  You've pushed that branch to the remote (`git push -u origin your-branch`).
    *   **How**:
        *   **GitHub**:
            1.  Go to repo > Click "Pull requests" tab > "New pull request".
            2.  Select `base` branch (target, e.g., `main`) and `compare` branch (your branch, e.g., `feature/login`).
            3.  Click "Create pull request".
            4.  Fill template: **Meaningful Title**, **Detailed Description** (What? Why? How? Screenshots? Fixes #123?).
        *   **GitLab**:
            1.  Go to repo > Click "Merge requests" > "New merge request".
            2.  Select `source branch` (your branch) and `target branch` (e.g., `main`).
            3.  Click "Compare branches and continue".
            4.  Fill template: Title, Description, Assignees, Reviewers, Labels, Milestone.
        *   **Bitbucket**:
            1.  Go to repo > Click "Create pull request" (top right).
            2.  Select `Source branch` (your branch) and `Destination branch` (e.g., `main`).
            3.  Fill template: Title, Description, Reviewers, Build status checks.
    *   **Best Practices for Creation**:
        *   **Small PRs/MRs**: 200-500 lines max. Easier to review, faster feedback, lower risk.
        *   **Atomic Commits**: Each commit should do one logical thing. PR should tell a clear story.
        *   **Clear Title/Description**: "Fix login timeout bug" > "Bug fix". Explain *impact* and *context*.
        *   **Link Issues**: "Closes #123" or "Related to PROJ-456" (auto-links/closes issues).
        *   **Check CI Status**: Ensure pipelines pass *before* requesting review (if possible).

*   **Reviewing PRs/MRs**
    *   **The Goal**: Improve code quality, share knowledge, ensure alignment. **NOT** to find fault.
    *   **Best Practices**:
        *   **Be Timely**: Don't let PRs/MRs sit for days. Set team SLAs (e.g., "review within 24h").
        *   **Be Constructive**: Focus on the *code*, not the person. "**Consider** using `map` here for clarity" > "This is wrong".
        *   **Ask Questions**: "What was the reasoning behind this approach?" > "This is bad".
        *   **Focus on Substance**: Logic, security, performance, maintainability > minor style nits (use linters for that!).
        *   **Use Threaded Comments**: Platforms allow commenting on specific lines. Use this for context.
        *   **Request Changes vs. Approve**: Be clear if you need fixes (`Request Changes`) or if it's good (`Approve`).
        *   **Rubber Ducking**: Read the code *aloud* to yourself – often reveals issues.
        *   **Check CI Results**: Don't approve if tests are failing!
        *   **Respect Scope**: Don't request unrelated changes ("While you're here...").

*   **Requesting Changes and Approvals**
    *   **Requesting Changes**:
        *   **What it means**: "This PR/MR *cannot* be merged in its current state. Specific changes are required."
        *   **When to use**: Critical bugs, security flaws, major design issues, failing tests.
        *   **How**: Platform button ("Request changes" on GitHub/GitLab, "Needs work" in Bitbucket). **Always leave specific comments** explaining *what* and *why*.
    *   **Approvals**:
        *   **What it means**: "I've reviewed this, it meets standards, and I believe it's ready to merge *if* other requirements are met (CI, other reviewers)."
        *   **When to use**: Code is correct, tests pass, follows style, meets requirements. *Doesn't mean "merge now!"*.
        *   **How**: Platform button ("Approve" on GitHub/GitLab, "Approve" in Bitbucket).
    *   **Critical Nuance**: **Approval ≠ Merge**. Merging usually requires:
        *   All required reviewers to approve.
        *   Passing required CI checks.
        *   Branch protection rules satisfied.
        *   Final decision often rests with the PR/MR creator or a maintainer.

*   **Using GitHub/GitLab UI Effectively**
    *   **Key Features**:
        *   **File Diff View**: See *exactly* what changed (`+` added, `-` removed). **Use "Hide whitespace changes"** to avoid noise.
        *   **Inline Comments**: Click line number > Add comment. Start threads. Resolve threads when fixed.
        *   **Review Actions**: `Comment` (just discussion), `Approve`, `Request Changes`.
        *   **Commits Tab**: See the full commit history of the PR/MR branch.
        *   **Checks/CI Tab**: See status of automated pipelines (tests, builds, scans). **Click to see logs**.
        *   **Conversation Tab**: Top-level discussion (scope, design decisions).
        *   **"Squash and Merge" / "Rebase and Merge" / "Create Merge Commit"**: **Choose wisely!** (See "Merging PRs/MRs" below).
        *   **Draft PRs/MRs**: Mark as "Draft" (GitHub) / "WIP" (GitLab/Bitbucket) while still working. Prevents accidental review/merge.
        *   **Assignees/Reviewers**: Explicitly tag who needs to act.
        *   **Labels**: Categorize (e.g., `bug`, `enhancement`, `needs review`, `blocked`).
        *   **Linked Issues**: Automatically close issues via description (`Closes #123`).

---

#### **3. Collaborative Development Workflow**
*(Keeping Code in Sync & Resolving Conflicts)*

*   **Syncing with Upstream Repositories**
    *   **The Problem**: You forked `upstream/main`. Others commit to `upstream/main`. Your fork's `main` (and your feature branches based on it) become **out of date**. PRs from your outdated branch might have conflicts.
    *   **The Solution**: Regularly sync your *local* `main` and your *fork's* `main` with the *original* `upstream/main`.
    *   **How**:
        1.  **Add Upstream Remote** (Do this ONCE per clone):
            ```bash
            git remote add upstream https://github.com/original-owner/project.git
            # Verify: git remote -v
            ```
        2.  **Fetch Upstream Changes** (Doesn't alter your working files):
            ```bash
            git fetch upstream
            ```
        3.  **Sync Your Local `main` Branch**:
            ```bash
            git checkout main          # Switch to your local main
            git rebase upstream/main   # OR git merge upstream/main (Rebase preferred for clean history)
            ```
        4.  **Sync Your Fork's `main` Branch** (Push updated local main to *your* fork):
            ```bash
            git push origin main       # Updates YOUR fork's main branch
            ```
    *   **Why Rebase over Merge?** `git rebase upstream/main` replays *your* commits *on top* of the latest `upstream/main`, resulting in a **linear, clean history**. `git merge upstream/main` creates a merge commit, which can clutter history if done frequently. *Rebasing is generally preferred for syncing feature branches*.

*   **Keeping Fork Up to Date**
    *   **The Problem**: Your fork (`your-username/project`) on GitHub/GitLab/Bitbucket lags behind the original (`original-owner/project`).
    *   **The Solution**: Sync your fork's `main` branch using the steps above (`git fetch upstream` -> `git rebase` locally -> `git push origin main`). **This updates your *server-side fork*.**
    *   **Platform UI Shortcut (GitHub Only)**:
        *   Go to your fork's repo page.
        *   If upstream has new commits, you'll see a banner: "This branch is X commits behind original-owner:main."
        *   Click "Sync fork" > "Update branch". *This does a merge, not a rebase.* **Use CLI for rebase!**
    *   **Critical**: Syncing your fork's `main` **does NOT automatically update your feature branches**. You must rebase *each feature branch* onto the updated `main` (see below).

*   **Resolving Merge Conflicts in PRs**
    *   **What Happens**: When changes in the PR branch *and* the target branch (`main`) modify the *same lines* of the *same file*, Git cannot automatically combine them. Conflict!
    *   **Where Conflicts Occur**:
        *   **During PR Creation**: Platform might warn "This branch has conflicts that must be resolved".
        *   **During PR Update**: When target branch changes *after* PR creation but *before* merge.
        *   **During Local Rebase/Merge**: When syncing your branch (most common).
    *   **How to Resolve (The Correct Way - Locally)**:
        1.  **Sync Target Branch**: Ensure your *local* `main` is up-to-date with upstream (as above).
        2.  **Rebase Feature Branch onto Updated Main**:
            ```bash
            git checkout feature-branch
            git rebase main   # OR git rebase upstream/main
            ```
        3.  **Conflict Detected**: Git stops at the first conflict. Files show `<<<<<<<`, `=======`, `>>>>>>>` markers.
        4.  **Resolve Manually**:
            *   Open conflicted file(s).
            *   Decide which changes to keep (yours, theirs, or a combination).
            *   **Remove conflict markers** (`<<<<<<< HEAD`, `=======`, `>>>>>>> commit-hash`).
            *   Save the file.
        5.  **Mark Resolved & Continue**:
            ```bash
            git add <resolved-file>   # Stage the fixed file
            git rebase --continue     # Move to next conflict or finish
            ```
        6.  **Force Push**: Since you changed history (rebased), you *must* force push to your remote branch:
            ```bash
            git push origin feature-branch --force-with-lease
            ```
        7.  **PR Updates Automatically**: The platform sees the new commits and updates the PR diff.
    *   **Why NOT Resolve in UI?** Platform UIs (GitHub "Resolve conflicts" button) *can* be used, but:
        *   Limited to simple conflicts.
        *   No local testing possible after resolution.
        *   Risk of accidental bad resolution.
        *   **Best Practice: Always resolve conflicts locally** where you can test the code!
    *   **Preventing Conflicts**: Small, frequent PRs; sync `main` often; communicate with team about overlapping changes.

---

#### **4. Contributing to Open Source Projects**
*(The Fork & Pull Workflow)*

*   **Finding Projects**:
    *   **GitHub Explore**, **GitLab Explore**, **CodeTriage**, **Up For Grabs**, **First Timers Only**.
    *   Look for `good first issue` / `beginner` / `help wanted` labels.
    *   Check project's `CONTRIBUTING.md` file (MANDATORY reading!).
    *   Ensure license is compatible with your goals.

*   **Fork & Clone Workflow (Step-by-Step)**:
    1.  **Fork the Upstream Repo**: Click "Fork" on the project's GitHub/GitLab page. Creates `your-username/project`.
    2.  **Clone YOUR Fork Locally**:
        ```bash
        git clone git@github.com:your-username/project.git
        cd project
        ```
    3.  **Add Upstream Remote**:
        ```bash
        git remote add upstream https://github.com/original-owner/project.git
        git fetch upstream
        ```
    4.  **Create a Feature Branch** (Off `upstream/main`, NOT your fork's main!):
        ```bash
        git checkout -b fix/issue-123 upstream/main
        ```
    5.  **Make Changes**: Edit code, write tests, update docs.
    6.  **Commit Changes** (Small, atomic commits!):
        ```bash
        git add changed-file.js
        git commit -m "fix: correct login timeout logic (closes #123)"
        ```
    7.  **Push to YOUR Fork**:
        ```bash
        git push -u origin fix/issue-123
        ```
    8.  **Create PR to Upstream**:
        *   Go to `your-username/project` on GitHub.
        *   Platform detects your new branch -> Click "Compare & pull request".
        *   Set `base repository: original-owner/project`, `base: main`, `head repository: your-username/project`, `compare: fix/issue-123`.
        *   Fill description thoroughly. Link issue (`Closes #123`).
    9.  **Engage in Review**: Respond to comments, make fixes (commit & push to *same branch*), update PR.
    10. **PR Merged!**: Upstream maintainer merges. **Your fork is now behind again!**
    11. **Clean Up**:
        *   Delete local branch: `git branch -d fix/issue-123`
        *   Delete remote branch: `git push origin --delete fix/issue-123`
        *   Sync your fork's `main` (Optional but good practice): `git checkout main`, `git fetch upstream`, `git rebase upstream/main`, `git push origin main --force` (or use UI sync).

*   **Submitting Patches via PRs - OSS Etiquette**:
    *   **Read `CONTRIBUTING.md`**: Non-negotiable. Follow coding style, test requirements, commit message format.
    *   **Start Small**: Fix typos, improve docs before tackling big features.
    *   **Communicate Early**: Comment on the issue *before* coding to confirm approach.
    *   **Respect Maintainers**: They are volunteers. Be patient, polite, and responsive.
    *   **Write Tests**: OSS projects often require them. Show you care about quality.
    *   **Update Documentation**: If your change affects users, update docs.
    *   **Squash Commits**: Before merge, clean up your history (`git rebase -i`) unless maintainer prefers otherwise. One logical change = one commit.
    *   **Don't Take Rejection Personally**: Feedback is about the code, not you.

---

#### **5. Using Git in Teams**
*(Enforcing Quality & Safety)*

*   **Branch Protection Rules**
    *   **What**: Rules enforced by GitHub/GitLab/Bitbucket **on specific branches** (usually `main`, `develop`, release branches).
    *   **Why**: Prevent accidental or unsafe changes to critical branches.
    *   **Key Rules**:
        *   **Require Pull Request Before Merging**: *Mandatory*. No direct pushes to protected branch.
        *   **Require Status Checks to Pass**: CI pipelines (tests, builds) MUST succeed before merge.
        *   **Require Code Reviews**: Specify *minimum number* of approvals (e.g., 1-2). Can require specific reviewers/groups.
        *   **Require Branch to Be Up to Date Before Merging**: Forces PR creator to rebase/merge `main` before merge, reducing conflicts.
        *   **Include Administrators**: *Crucial!* Ensures even repo owners must follow the rules.
        *   **Restrict Pushing**: Only specific users/groups can push (often disabled since PRs are required).
        *   **Require Linear History**: Enforces `Rebase and Merge` or `Squash and Merge` (no merge commits).
        *   **Allow Force Pushes**: *Almost always disabled* on protected branches (destroys history).
        *   **Require Signed Commits**: For security/compliance (GPG signing).
    *   **Where to Set**: GitHub: Repo Settings > Branches > Branch protection rules. GitLab: Repo Settings > Repository > Protected Branches. Bitbucket: Repo Settings > Branch permissions.

*   **Required Reviewers**
    *   **What**: Specific people or teams **mandated** to approve a PR/MR before it can merge.
    *   **Why**: Ensure domain experts (backend, security, UI) review relevant changes. Distribute knowledge.
    *   **How**:
        *   **GitHub**: In PR creation/edit, add "Reviewers". Can set *code owners* (`.github/CODEOWNERS` file) to auto-assign reviewers based on file paths.
        *   **GitLab**: In MR creation/edit, add "Reviewers". Also supports `CODEOWNERS` file.
        *   **Bitbucket**: In PR creation/edit, add "Reviewers". Supports `CODEOWNERS`.
    *   **Best Practices**:
        *   Use `CODEOWNERS` for scalability (avoids manual assignment).
        *   Don't over-assign (slows down process). Target 1-2 key reviewers.
        *   Respect reviewer bandwidth (don't tag 5 people for a tiny change).

*   **CI/CD Integration with PRs**
    *   **What**: Automated pipelines (Continuous Integration / Continuous Delivery) **triggered by PRs/MRs**.
    *   **Why**: Catch bugs *before* code hits `main`. Ensure quality gate.
    *   **How It Works**:
        1.  You push to a PR branch (`feature/login`).
        2.  Platform (GitHub Actions, GitLab CI, Bitbucket Pipelines) detects the push.
        3.  Pipeline runs defined steps:
            *   `build`: Compile code.
            *   `test`: Run unit/integration tests.
            *   `lint`: Check code style.
            *   `scan`: Security (SAST), license checks.
            *   `deploy (staging)`: Optional - deploy to test environment.
        4.  Pipeline status (✅/❌) is reported **directly on the PR/MR**.
        5.  **Branch Protection** can *require* specific checks to pass (✅) before merging.
    *   **Critical Benefit**: **Prevents broken code from merging**. A failing test = blocked merge.

*   **Team Branching Policies**
    *   **What**: **Agreed-upon strategies** for how branches are created, used, and merged within a team.
    *   **Why**: Avoid chaos, ensure everyone understands the workflow, enable safe collaboration.
    *   **Common Strategies**:
        *   **GitHub Flow (Simple & Popular)**:
            *   `main` branch is **always deployable**.
            *   Create **feature branches** directly from `main`.
            *   Work on feature branch -> Push -> Open PR to `main`.
            *   PR reviewed, CI passes -> **Merge** (Squash/Rebase preferred) -> `main` updated -> **Deploy**.
            *   *Pros*: Simple, fast, great for web apps with frequent deploys. *Cons*: Less structured for complex releases.
        *   **GitFlow (More Structured)**:
            *   `main` = Production releases (tagged).
            *   `develop` = Integration branch for features (next release).
            *   `feature/*` branches -> Merge into `develop`.
            *   `release/*` branches: Stabilize `develop` for release (merge into `main` & `develop`).
            *   `hotfix/*` branches: Fix `main` directly (merge into `main` & `develop`).
            *   *Pros*: Clear separation for releases/hotfixes. *Cons*: Complex, `develop` can become unstable, merge overhead. *Less common now*.
        *   **Trunk-Based Development (Advanced)**:
            *   `main` (trunk) is **always stable and deployable**.
            *   **Short-lived feature branches** (hours, not days). *Or* direct commits to `main` behind feature flags.
            *   Heavy reliance on **CI, automated testing, and feature flags**.
            *   *Pros*: Minimizes merge hell, enables continuous delivery. *Cons*: Requires strong engineering culture/practices.
    *   **Key Policy Decisions**:
        *   **Branch Naming Conventions**: `feat/`, `fix/`, `docs/`, `chore/`? (e.g., `feat/user-auth`).
        *   **PR Size Limits**: Max lines/commits per PR.
        *   **Review SLAs**: Max time to review a PR.
        *   **Merge Strategies**: Enforce `Squash and Merge`? `Rebase and Merge`? Why?
        *   **Commit Message Standards**: Conventional Commits (`feat:`, `fix:`, etc.)?
        *   **Who Merges?**: PR creator? Reviewer? Maintainer?
        *   **Hotfix Process**: How to handle critical production bugs?
    *   **Document It!**: Put your branching policy in `CONTRIBUTING.md` or a team wiki. **Everyone must follow it.**

---

### **Key Takeaways & Best Practices Summary**

1.  **SSH > HTTPS**: Set up SSH keys properly for seamless, secure access.
2.  **Fork for OSS, Clone Directly for Teams**: Understand the difference.
3.  **PRs/MRs are Sacred**: Small, well-described, linked to issues, passing CI.
4.  **Review is Collaborative**: Be constructive, timely, and focus on quality.
5.  **Sync Early, Sync Often**: Rebase your feature branches onto updated `main` *before* conflicts happen. **Resolve conflicts locally**.
6.  **Branch Protection is Non-Negotiable**: Enforce PRs, reviews, and CI for critical branches.
7.  **CI/CD is Your Safety Net**: PRs must trigger automated checks; merges require green builds.
8.  **Define & Document Your Workflow**: GitHub Flow is a great starting point for most teams. Consistency is key.
9.  **OSS Contribution = Etiquette**: Read `CONTRIBUTING.md`, communicate, start small, be patient.
10. **Master the CLI**: UIs are helpful, but deep Git understanding requires command-line proficiency (`rebase`, `fetch`, `push --force-with-lease`).
