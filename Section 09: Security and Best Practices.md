### 🔐 **1. Securing Git Repositories: The Foundation**
*   **Why it Matters:** Git repositories often contain source code, configuration, and potentially sensitive data. Unsecured repos are prime targets for attackers (data theft, supply chain attacks, ransomware).
*   **Core Principle:** **Defense-in-Depth.** Relying on a single security layer (like just a password) is insufficient. Implement multiple overlapping controls.

---

### 🔑 **2. SSH Key Management (Secure Access to Repos)**
*   **What it is:** SSH (Secure Shell) uses public-key cryptography for authentication instead of passwords. You have a *private key* (kept secret on your machine) and a *public key* (added to your Git server account - GitHub, GitLab, etc.).
*   **Why Use SSH?**
    *   **Stronger than Passwords:** Immune to brute-force password guessing.
    *   **Convenience:** Once set up, no need to enter username/password for every operation (especially with `ssh-agent`).
    *   **Mandatory for Some Actions:** Often required for pushing to protected branches or interacting with CI/CD systems.
*   **Critical Best Practices:**
    *   **Use Strong Keys:** Prefer **Ed25519** (`ssh-keygen -t ed25519 -C "your_email@example.com"`) over RSA (especially <2048-bit). Ed25519 is faster and more secure.
    *   **ALWAYS Use a Passphrase:** Protects your private key if your machine is compromised. `ssh-keygen` will prompt you. *Never create a passwordless key for production access.*
    *   **Store Keys Securely:**
        *   Private keys (`id_ed25519`, `id_rsa`) **MUST** reside in `~/.ssh/` (or `%USERPROFILE%\.ssh\` on Windows).
        *   Permissions are CRITICAL: `chmod 700 ~/.ssh` (drwx------), `chmod 600 ~/.ssh/id_*` (-rw-------). Git servers often reject keys with loose permissions.
    *   **Use `ssh-agent`:** This background program holds your decrypted private keys in memory *after* you enter the passphrase once per session. Start it: `eval $(ssh-agent -s)`, then add your key: `ssh-add ~/.ssh/id_ed25519`. Avoid `ssh-add -K` (macOS keychain) unless you fully understand the security implications.
    *   **Rotate Keys Periodically:** Treat SSH keys like passwords. Rotate them (generate new key pair, replace public key on Git server) every 6-12 months or after a security incident.
    *   **Revoke Compromised Keys IMMEDIATELY:** Remove the public key from *all* Git server accounts (GitHub, GitLab, Bitbucket, etc.) it was added to. **This is non-negotiable.**
    *   **Never Commit Private Keys:** `.gitignore` should *always* exclude `~/.ssh/` patterns. Accidentally committing `id_rsa` is a catastrophic breach.

---

### 🔒 **3. Two-Factor Authentication (2FA)**
*   **What it is:** Requires **two distinct proofs** of identity to log in: something you *know* (password/PAT) + something you *have* (phone app code, security key).
*   **Why it's Essential:**
    *   **Mitigates Password/PAT Theft:** Even if your password or PAT is stolen (phishing, keylogger, database breach), the attacker *cannot* log in without the second factor.
    *   **Industry Standard:** Required by most enterprise security policies and compliance frameworks (SOC 2, HIPAA, etc.).
    *   **Protects Account Recovery:** Secures the "forgot password" flow.
*   **Implementation:**
    *   **Enable on EVERY Git Service:** GitHub, GitLab, Bitbucket, Azure DevOps, etc. Go to your account **Security Settings**.
    *   **Preferred Methods:**
        *   **Authenticator Apps (TOTP):** Google Authenticator, Authy, Microsoft Authenticator. *Most common and recommended.* Generate a QR code during setup.
        *   **Security Keys (FIDO2/U2F):** YubiKey, Titan Security Key. *Strongest method* (phishing-resistant). Highly recommended for critical accounts.
    *   **Avoid SMS (if possible):** SMS is vulnerable to SIM swapping attacks. Use only if no other option exists.
    *   **Backup Codes:** **SAVE THESE SECURELY** (password manager, physical safe). They are your ONLY way in if you lose your 2FA device. Treat them like passwords.
*   **Critical Note:** 2FA protects **web login and API access** (including Git over HTTPS). It does *not* directly secure Git operations using SSH keys (SSH keys *are* the authentication mechanism). However, enabling 2FA on your account is still mandatory for overall account security.

---

### 🔑 **4. Personal Access Tokens (PATs)**
*   **What it is:** A long, randomly generated string (like `ghp_abc123xyz...`) used **instead of your password** for Git operations over HTTPS or API access. Scoped to specific permissions.
*   **Why Use PATs (vs. Passwords):**
    *   **Granular Permissions:** Limit token scope (e.g., `repo` for code, `read:org` for org info, `delete_repo` - *use sparingly!*). Passwords grant *full* account access.
    *   **Revocable:** Can be deleted instantly if compromised. Changing your password invalidates *all* sessions (inconvenient).
    *   **Expiration:** Can set short lifespans (e.g., 30 days, 90 days, 180 days - check your provider's max). Passwords don't expire by default.
    *   **Required by Many Platforms:** GitHub deprecated password auth for Git operations in 2021. GitLab, Bitbucket strongly encourage PATs.
*   **Best Practices:**
    *   **ALWAYS Use PATs for HTTPS Git Operations:** Configure Git: `git config credential.helper store` (caches securely) or use OS keychain. Enter PAT as password.
    *   **Minimal Scope:** Grant *only* the permissions absolutely necessary (e.g., `repo` for read/write code, `read:org` if needed). **NEVER use `admin:org` or `delete_repo` scopes for routine Git operations.**
    *   **Set Expiration:** Use the shortest practical expiration (e.g., 30-90 days). Rotate tokens regularly.
    *   **Descriptive Names:** Name tokens clearly (`laptop-work-2023-10`, `jenkins-ci-token`). Makes revocation easier.
    *   **Treat Like Passwords:** **NEVER commit PATs, hardcode them in scripts, or share them.** Store in secure vaults (1Password, Bitwarden, HashiCorp Vault) or CI/CD secrets managers.
    *   **Use Fine-Grained PATs (GitHub):** Newer tokens allow even more precise repo/permission control. Prefer these over Classic PATs.

---

### 🚫 **5. Revoking Compromised Tokens**
*   **Why Immediate Action is Critical:** A leaked PAT is equivalent to a compromised password with potentially broader permissions. Attackers can:
    *   Steal source code
    *   Inject malicious code
    *   Delete repositories
    *   Access other linked services
*   **Steps to Revoke:**
    1.  **Identify:** Determine *which* token was compromised (check descriptive names!).
    2.  **Revoke IMMEDIATELY:**
        *   **GitHub:** Settings -> Developer Settings -> Personal Access Tokens (Classic) OR Fine-grained tokens -> Find token -> Click `...` -> `Revoke`.
        *   **GitLab:** Settings -> Access Tokens -> Find token -> Click `Revoke`.
        *   **Bitbucket:** Settings -> App Passwords (older) OR Personal Access Tokens -> Find token -> `Delete`.
        *   **Azure DevOps:** User Settings (top right avatar) -> Personal Access Tokens -> Find token -> `Revoke`.
    3.  **Rotate:** Generate a **new** token with the *minimum necessary scope* and update all systems using the old token (CI/CD, local Git config, scripts).
    4.  **Audit:** Check repository access logs (if available) for suspicious activity *during* the token's validity period. Check commit history for unauthorized changes.
    5.  **Notify:** If the token had high privileges or accessed sensitive repos, inform your security team.
*   **Prevention is Key:** Using short-lived tokens, minimal scope, and secure storage drastically reduces the blast radius of a leak.

---

### ✍️ **6. GPG Signing Commits and Tags**
*   **What it is:** Using **GnuPG (GPG)** to cryptographically sign commits and tags. Proves:
    *   **Authenticity:** The commit *was* made by the claimed author (you).
    *   **Integrity:** The commit *has not been altered* since it was signed.
*   **Why it Matters:**
    *   **Prevents Impersonation:** Stops attackers from making commits *as you* if they compromise your Git server password/PAT.
    *   **Builds Trust:** Critical for open-source projects (users verify maintainers' commits). Required by some enterprises/compliance.
    *   **Audit Trail:** Provides verifiable proof of who made changes and when.
*   **How it Works:**
    1.  You generate a GPG key pair (private/public).
    2.  You add your **public key** to your Git server account (GitHub, GitLab, etc.).
    3.  When you commit/tag, Git uses your **private key** to create a digital signature.
    7.  Git servers (and `git log --show-signature`) use your **public key** to verify the signature.

---

### 🔐 **7. Setting Up GPG Keys**
*   **Step 1: Install GPG**
    *   Linux: `sudo apt-get install gnupg` (Debian/Ubuntu) or `sudo yum install gnupg` (RHEL/CentOS)
    *   macOS: `brew install gnupg` (via Homebrew) or use built-in `/usr/bin/gpg`
    *   Windows: Install [Gpg4win](https://www.gpg4win.org/)
*   **Step 2: Generate a Key Pair**
    ```bash
    gpg --full-generate-key
    ```
    *   Select `(1) RSA and RSA (default)`
    *   **Key Size: 4096 bits** (Minimum. Avoid 1024/2048 if possible).
    *   Set **Key Expiration** (e.g., 1y, 2y - allows planned rotation).
    *   **Real Name:** Your name *exactly* as in Git commits (`git config user.name`).
    *   **Email:** Your email *exactly* as in Git commits (`git config user.email`). **MUST match the email associated with your Git server account.**
    *   **Passphrase:** **STRONG and MEMORABLE.** Protects your private key. *Crucial!*
*   **Step 3: Find Your Key ID**
    ```bash
    gpg --list-secret-keys --keyid-format LONG
    ```
    Look for `sec   rsa4096/XXXXXXXXYYYYYYYY` - `XXXXXXXXYYYYYYYY` is your **Key ID** (last 16 chars of fingerprint).
*   **Step 4: Configure Git to Use GPG**
    ```bash
    git config --global user.signingkey XXXXXXXXYYYYYYYY  # Your Key ID
    git config --global gpg.program gpg  # Ensure correct path (usually auto-detected)
    ```
*   **Step 5: Add Public Key to Git Server**
    *   Export Public Key: `gpg --armor --export XXXXXXXXYYYYYYYY`
    *   Copy the output (starts with `-----BEGIN PGP PUBLIC KEY BLOCK-----`).
    *   **GitHub:** Settings -> SSH and GPG keys -> New GPG key -> Paste.
    *   **GitLab:** Settings -> GPG Keys -> Paste.
    *   **Bitbucket:** Not natively supported (use commit signing via pipelines or external verification).

---

### ✅ **8. `git config commit.gpgsign true`**
*   **What it Does:** Sets the `commit.gpgsign` option to `true` **globally** (for all repos on your machine). This means **every commit you make will be signed by default** using the GPG key specified in `user.signingkey`.
*   **Why Use It:** Ensures you *never accidentally make an unsigned commit*. Critical for maintaining a verifiable history.
*   **How to Set:**
    ```bash
    git config --global commit.gpgsign true
    ```
*   **Per-Repo Override:** If you need unsigned commits in a *specific* repo (rare, e.g., internal test repo), go into that repo and run:
    ```bash
    git config commit.gpgsign false
    ```
*   **Verification:** When you commit, you'll be prompted for your GPG key **passphrase** (if using `gpg-agent`, only once per session). The commit will show as "Verified" on GitHub/GitLab.

---

### 🔍 **9. Verifying Signed Commits**
*   **Why Verify:** To trust that commits/tags truly come from verified authors and haven't been tampered with.
*   **How to Verify:**
    *   **Git CLI (Single Commit):**
        ```bash
        git log --show-signature -1  # Shows signature status for the latest commit
        ```
        Look for `Good signature from "Your Name <email>"` and `gpg: Signature made ...` **AND** `gpg:                 using RSA key XXXXXXXXYYYYYYYY`. A `BAD signature` means tampering or wrong key.
    *   **Git CLI (All Commits in Range):**
        ```bash
        git verify-commit <commit-hash>  # Verify a specific commit
        git verify-tag <tag-name>        # Verify a specific tag
        ```
    *   **GitHub/GitLab Web UI:** Verified commits/tags show a **"Verified" badge** (green checkmark). Unverified show "Unverified" or nothing. *This relies on the platform having your public key.*
*   **Critical Note:** Verification **only works** if:
    1.  The committer's public key is known/trusted by the verifier (added to their Git server account or local keyring).
    2.  The commit was signed with the *correct* private key matching that public key.
    3.  The commit hasn't been altered since signing.

---

### ⚠️ **10. Avoiding Sensitive Data in Git: The Golden Rule**
*   **The Cardinal Sin:** **NEVER commit secrets to Git history.** Once committed, they are *extremely hard* to fully remove (see Section 14). History is permanent by design.
*   **What NOT to Commit:**
    *   Passwords, API Keys, Secret Keys (AWS, GCP, Azure, Stripe, etc.)
    *   `.env` files (containing secrets)
    *   Configuration files with secrets (`config.yml`, `secrets.json`, `application.properties`)
    *   SSH Private Keys, SSL Certificates (with private keys)
    *   Database connection strings with passwords
    *   Any personally identifiable information (PII) or sensitive customer data
*   **Why it's Catastrophic:**
    *   **Public Repos:** Instantly exposed to the entire internet. Automated bots scan GitHub constantly for secrets.
    *   **Private Repos:** Still vulnerable to insider threats, compromised accounts, misconfigured permissions, or accidental public exposure. Secrets *leak* from private repos frequently.
    *   **History is Permanent:** Even if you delete the file later, the secret remains in the commit history. `git blame` reveals it.

---

### 🧹 **11. Using `.gitignore` and Pre-Commit Hooks**
*   **`.gitignore` (The First Line of Defense):**
    *   **What it is:** A file in your repo root that tells Git **which files/directories to NEVER track**, even if they exist locally.
    *   **Critical Patterns:**
        ```gitignore
        # Environment Variables
        .env
        *.env
        env/
        .env.local
        .env.*.local

        # Secrets / Keys
        *.pem
        *.key
        *.crt
        *.jks
        *.keystore
        id_rsa
        id_ed25519

        # Build/Dependency Artifacts (often contain secrets)
        node_modules/
        vendor/
        target/
        build/
        dist/
        *.exe
        *.dll
        *.so

        # IDE/OS Specific
        .idea/
        .vscode/
        .DS_Store
        Thumbs.db
        ```
    *   **Best Practices:**
        *   **Start Early:** Create `.gitignore` *before* initializing the repo (`git init`).
        *   **Use Templates:** Leverage [gitignore.io](https://www.toptal.com/developers/gitignore) for language/framework-specific templates.
        *   **Review Carefully:** Don't blindly copy templates. Understand what you're ignoring.
        *   **Commit `.gitignore`:** It's crucial for *all* collaborators.
        *   **It's NOT Retroactive:** `.gitignore` only prevents *new* files from being tracked. It **DOES NOT** hide files already in history! (See Section 14).
*   **Pre-Commit Hooks (Proactive Scanning):**
    *   **What it is:** Scripts that run **automatically** *before* a commit is finalized (`git commit`). Can inspect staged files and **BLOCK** the commit if problems are found.
    *   **Why Essential:** Catches secrets *before* they enter history. `.gitignore` prevents tracking, but pre-commit hooks prevent *accidental staging* of ignored files or secrets in non-ignored files.
    *   **Implementation via `pre-commit` Framework (Highly Recommended):**
        1.  Install: `pip install pre-commit`
        2.  Create `.pre-commit-config.yaml` in repo root:
            ```yaml
            repos:
            -   repo: https://github.com/pre-commit/pre-commit-hooks
                rev: v4.4.0
                hooks:
                -   id: check-yaml
                -   id: check-json
                -   id: end-of-file-fixer
                -   id: trailing-whitespace
            -   repo: https://github.com/awslabs/git-secrets
                rev: 1.3.0
                hooks:
                -   id: git-secrets
            -   repo: https://github.com/zricethezav/gitleaks
                rev: v8.16.1
                hooks:
                -   id: gitleaks
            ```
        3.  Install Hooks: `pre-commit install` (Sets up the Git hook script)
        4.  **How it Works:** On `git commit`, the framework runs all configured hooks. `git-secrets` and `gitleaks` will scan *staged* files for secrets. If found, the commit is **ABORTED** with an error message showing the leaked secret.
    *   **Key Advantage:** Catches secrets embedded *within code/config files* (e.g., `api_key = "abc123"` in a `.js` file), which `.gitignore` cannot prevent.

---

### 🛠️ **12. Tools: `git-secrets`, `pre-commit`, `gitleaks`**
*   **`pre-commit` Framework:**
    *   **Role:** The *orchestrator*. Manages installing, updating, and running multiple hooks (like `git-secrets`, `gitleaks`, linters, formatters).
    *   **Why Use It:** Standardizes hook management across projects/teams. Easy configuration via `.pre-commit-config.yaml`.
*   **`git-secrets` (AWS Tool):**
    *   **Role:** Prevents committing *known patterns* of secrets (AWS keys, generic secrets).
    *   **How it Works:**
        *   Scans staged files against a set of **default regex patterns** (e.g., `AWS_ACCESS_KEY_ID`, `-----BEGIN RSA PRIVATE KEY-----`).
        *   Can add **custom patterns** (e.g., `MYAPP_API_SECRET = "([a-zA-Z0-9]{40})"`).
        *   Can register **allowed secrets** (false positives) via `git secrets --add --allowed`.
    *   **Strengths:** Simple, fast, integrates well with `pre-commit`. Good for common patterns.
    *   **Limitations:** Regex-based, can have false negatives (obfuscated secrets) or false positives. Less comprehensive scanning than `gitleaks`.
*   **`gitleaks` (Open Source Scanner):**
    *   **Role:** Advanced secret detection using sophisticated scanning techniques.
    *   **How it Works:**
        *   Uses **hundreds of built-in regex patterns** (more extensive than `git-secrets`).
        *   Employs **entropy analysis** to detect high-entropy strings (common in API keys) even if they don't match a specific pattern.
        *   Can scan **entire commit history** (`gitleaks detect --log-branch`), not just staged files.
        *   Generates detailed reports (JSON, CSV, stdout).
    *   **Strengths:** More accurate detection (lower false negative rate), better at finding novel/obfuscated secrets, history scanning.
    *   **Limitations:** Slightly slower than `git-secrets`, higher chance of false positives (needs tuning). Requires configuration (`gitleaks.toml`).
*   **Synergy:** Use **BOTH** `git-secrets` (via `pre-commit`) for fast pre-commit blocking *and* `gitleaks` (via `pre-commit` *and* scheduled CI/CD scans) for deeper historical and entropy-based scanning. They complement each other.

---

### 🧹 **13. Rewriting History to Remove Secrets (The Nuclear Option)**
*   **When is it Needed?** **ONLY** if a secret was **accidentally committed and pushed to a shared branch** (like `main`/`master`). **This should be a last resort** – prevention (Sections 10-12) is vastly preferable.
*   **Why it's Dangerous:**
    *   **Breaks History:** Changes commit SHAs. **ALL collaborators MUST re-clone or reset their local repos.** Causes massive disruption.
    *   **Doesn't Erase from All Copies:** Secrets might still exist in:
        *   Forks of the repo
        *   CI/CD build caches
        *   GitHub/GitLab download caches (though platforms try to purge)
        *   Anyone's local clone who hasn't reset
    *   **Revocation is STILL Mandatory:** **ALWAYS revoke the leaked secret FIRST** (Section 5). Rewriting history *does not* invalidate the secret itself!
*   **Tools (Choose ONE):**
    *   **`git filter-repo` (STRONGLY PREFERRED - Modern Standard):**
        *   **Why:** Faster, safer, more powerful, actively maintained successor to `filter-branch` and BFG. **Use this by default.**
        *   **Installation:** `pip install git-filter-repo` (Recommended) or download script.
        *   **Basic Usage (Remove a File):**
            ```bash
            # 1. Clone a FRESH copy (DON'T WORK ON YOUR MAIN CLONE!)
            git clone https://github.com/yourusername/yourrepo.git
            cd yourrepo
            # 2. Remove the file EVERYWHERE in history
            git filter-repo --force --invert-paths --path path/to/secret/file.txt
            # 3. (Optional) Remove all .env files
            git filter-repo --force --invert-paths --path-glob '*.env'
            # 4. (CRITICAL) Force push cleaned history (DESTROYS REMOTE HISTORY)
            git push origin --force --all
            git push origin --force --tags
            ```
        *   **Advanced (Remove Content within Files - Use with EXTREME Caution):**
            ```bash
            git filter-repo --force --replace-text <(echo "old-secret-string::new-value-or-empty")
            ```
            *   **WARNING:** This is error-prone. `new-value-or-empty` should often be a *placeholder* (like `REMOVED_SECRET`) or empty string. **Test extensively on a dummy repo first!** Can corrupt files.
    *   **BFG Repo-Cleaner (Legacy - Use only if `filter-repo` isn't feasible):**
        *   **Why:** Simpler CLI for common tasks (deleting files), but less flexible and slower than `filter-repo`.
        *   **Installation:** Download JAR from [https://rtyley.github.io/bfg-repo-cleaner/](https://rtyley.github.io/bfg-repo-cleaner/)
        *   **Basic Usage:**
            ```bash
            # 1. Clone a FRESH copy
            git clone --mirror https://github.com/yourusername/yourrepo.git
            cd yourrepo.git
            # 2. Delete a specific file
            java -jar bfg.jar --delete-files id_rsa
            # 3. Delete all .env files
            java -jar bfg.jar --delete-files '*.env'
            # 4. Run the cleaner
            git reflog expire --expire=now --all && git gc --prune=now --aggressive
            # 5. Force push
            git push --force
            ```
*   **Force-Pushing Cleaned History (CAUTION!):**
    *   **`git push --force` / `git push --force-with-lease`:** This **overwrites the remote branch history** with your cleaned local history.
    *   **`--force-with-lease` is SAFER:** Only force-pushes if the remote branch hasn't changed since you last fetched. Prevents accidentally overwriting *others'* new commits. **ALWAYS prefer `--force-with-lease`.**
    *   **Impact:** **ALL collaborators must run:**
        ```bash
        git fetch origin
        git reset --hard origin/main  # Replace 'main' with your default branch
        ```
        Their local history will be destroyed and replaced. **Communicate this widely BEFORE force-pushing!**
    *   **GitHub/GitLab Protection:** If your default branch (`main`) is **protected** (common), you **MUST** temporarily disable branch protection rules (like "Require status checks" or "Require pull request reviews") *before* force-pushing, then re-enable them immediately after. **Document this step!**

---

### 📌 **Critical Summary & Action Plan**

1.  **PREVENT (Most Important!):**
    *   **Enable 2FA** on *all* Git accounts.
    *   **Use SSH Keys** (Ed25519, passphrase-protected) or **PATs** (short-lived, minimal scope) for Git access.
    *   **Configure `.gitignore`** aggressively from day one.
    *   **Implement Pre-Commit Hooks** (`pre-commit` + `git-secrets` + `gitleaks`) to BLOCK secrets *before* commit.
    *   **Sign Commits** (`git config commit.gpgsign true` + GPG key setup).
2.  **DETECT:**
    *   Run `gitleaks detect` in **CI/CD pipelines** to scan history regularly.
    *   Monitor security alerts from GitHub/GitLab (they scan for secrets).
3.  **RESPOND (If Breach Occurs):**
    *   **REVOKE** the compromised secret/token **IMMEDIATELY** (Section 5).
    *   **ONLY IF** secret is in pushed history: Use `git filter-repo` on a **fresh clone**, **force-push with lease**, and **communicate widely** to collaborators. **REVOCATION IS STILL REQUIRED.**
    *   **Audit** for damage.

**Remember:** Git is a *distributed* ledger. **Once something is pushed, it's potentially everywhere.** Prevention through tooling (`pre-commit`, `gitleaks`) and strict practices (PATs, 2FA, GPG) is infinitely cheaper and safer than cleanup. Use history rewriting only as an absolute last resort after revocation.
