---
name: secrets-setup
description: "Interactive guide for setting up secure secret storage on your workstation. Detects your OS, asks about your environment (Vault access, tools used), and configures the recommended secret storage method with JIT fetching. Use when setting up a new machine, rotating to a better secrets workflow, or onboarding a teammate."
allowed-tools: read_file file_search grep_search apply_patch create_file list_dir run_in_terminal
metadata:
  version: "0.1.0"
  source: "@tyler.given/skill-secrets-setup"
  supported_languages: "bash, zsh, powershell"
  supported_frameworks: "github-copilot-skills"
  supported_operating_systems: "macos, linux, windows"
  categories: "security, tooling, onboarding"
  tags: "secrets, credentials, keychain, keyring, vault, gcm, git-credential-manager, security, setup, onboarding, new-machine"
user-invocable: true
---

# secrets-setup

Interactive guide for setting up secure secret storage on your workstation.

## When to use

- Setting up a new workstation
- Switching from plaintext secrets to secure storage
- Helping a teammate configure their dev environment
- Rotating secrets or changing secret storage method

## Decision Protocol

Follow these steps interactively with the user, gathering information before recommending and configuring a solution.

### Step 1: Detect Environment

Automatically discover the user's OS and available tooling:

1. **Detect OS** — run `uname -s` or inspect `$OSTYPE` to determine macOS, Linux, or Windows.
2. **Check available tools:**
   - `security` (macOS Keychain CLI)
   - `keyctl` (Linux kernel keyring)
   - `git credential-manager` (cross-platform GCM)
   - `vault` (HashiCorp Vault CLI)
3. **Check for existing Vault config** — look for `~/.config/vault-agent/`.
4. **Scan for plaintext secrets** — run:
   ```bash
   grep -r 'export.*TOKEN\|export.*SECRET\|export.*KEY' ~/.zshrc ~/.bashrc ~/.bash_profile 2>/dev/null
   ```
   Report any matches to the user (these will be migrated later).

Present findings to the user before continuing.

### Step 2: Ask User Context

Ask the user these questions and wait for answers:

1. **Do you have access to a HashiCorp Vault instance?** (Yes / No)
2. **What secrets do you need to store?** (Dev tokens, DB credentials, SSH keys, all of the above)
3. **Do any of your tools require persistent `$ENV_VAR` exports?** (npm, Docker, CI tools — Yes / No)

### Step 3: Recommend Method

Based on the answers from Steps 1 and 2, recommend **one** of the following:

| Priority | Method | When to recommend |
|----------|--------|-------------------|
| 1 | **Vault + OS fallback** | User has Vault access — best security, centralized management |
| 2 | **OS-native only** | No Vault — use macOS Keychain, Linux keyring, or Windows Credential Manager |
| 3 | **Git Credential Manager** | Cross-platform, simplest setup, good for Git-centric workflows |

Explain the recommendation and get user confirmation before proceeding.

### Step 4: Configure

Execute the setup for the chosen method:

1. **Install dependencies** — install any needed tools:
   - Linux: `keyutils` package for `keyctl`
   - Any OS: `vault` CLI if Vault method chosen
   - Any OS: `git-credential-manager` if GCM method chosen

2. **Create shell helper functions** — generate the appropriate functions based on OS and chosen method:
   - `_keychain_get` — macOS Keychain retrieval
   - `_keyring_get` — Linux keyring retrieval
   - `_gcm_get` — Git Credential Manager retrieval
   - `_vault_get` — HashiCorp Vault retrieval

3. **Create `_secret_get` resilient wrapper** — a single entry-point function that tries the configured backends in priority order with fallback:
   ```bash
   _secret_get() {
     local key="$1"
     # Try backends in configured priority order; return first success
   }
   ```

4. **Add convenience aliases** for common tools that need secrets:
   - `gl` — authenticated GitHub CLI
   - `npmp` — npm publish with token
   - `gcurl` — curl with auth header
   - Other aliases as appropriate for the user's tools

5. **Migrate plaintext secrets** — for any secrets found in Step 1:
   - Store each one in the chosen backend
   - Replace the `export` line with a JIT `_secret_get` call
   - Comment out (do not delete) the original line as a backup

6. **Add source line** to the user's shell profile (`.zshrc`, `.bashrc`, etc.) so functions load on every new shell.

### Step 5: Verify

Confirm everything works:

1. **Round-trip test** — store a test secret, retrieve it, and confirm the value matches.
2. **Shell function test** — open a new shell (or `source` the profile) and verify functions load without errors.
3. **Vault status** — if Vault was configured, run `vault status` and confirm connectivity.

Report results to the user. If any step fails, diagnose and fix before continuing.

### Step 6: Educate

Wrap up by teaching the user what was done and how to maintain it:

1. **JIT vs export tradeoff** — explain that JIT fetching (`_secret_get` at call time) avoids secrets sitting in environment variables but adds a small latency cost per invocation. Persistent `export` is acceptable only when a tool strictly requires it.
2. **Reference documentation** — point to the secrets management best practices for the full guide.
3. **Rotation reminder** — remind the user to rotate secrets on a regular schedule.

## Authoritative Sources

The guidance in this skill is derived from the following official documentation:

- **macOS Keychain** — [Apple Security CLI (`security`)](https://developer.apple.com/documentation/security/keychain_services) and `man security`
- **Linux kernel keyring** — [Linux `keyctl` man page](https://man7.org/linux/man-pages/man1/keyctl.1.html) and [kernel keyring docs](https://www.kernel.org/doc/html/latest/security/keys/core.html)
- **HashiCorp Vault** — [Vault documentation](https://developer.hashicorp.com/vault/docs) and [Vault Agent](https://developer.hashicorp.com/vault/docs/agent-and-proxy/agent)
- **Git Credential Manager** — [GCM documentation](https://github.com/git-ecosystem/git-credential-manager/blob/main/README.md)
- **Windows Credential Manager** — [Microsoft Credential Manager docs](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication)

## See Also

- [tyler555g/best-practices](https://github.com/tyler555g/best-practices) — secrets management best practices
