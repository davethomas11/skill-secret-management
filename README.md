# skill-secrets-setup

Copilot skill for secure secret management using an **inject model** — the agent orchestrates operations but **never sees secret values**.

## Architecture

```
Agent (LLM context)          │  Script (subprocess)
─────────────────────────────┼───────────────────────────────
"store GITLAB_TOKEN"    ──►  │  read -rsp → backend write
"inject KEY -- cmd"     ──►  │  backend read → env KEY=val cmd
"list"                  ──►  │  key names only → stdout
"exists KEY"            ──►  │  exit code 0/1
"delete KEY"            ──►  │  backend delete
                             │
  Agent sees: exit codes,    │  Script sees: secret values
  key names, cmd output      │  (scoped, never printed)
```

## Install

```bash
git clone https://github.com/tyler555g/skill-secrets-setup.git ~/.copilot/skills/secrets-setup
```

## Operations

| Op | Command | What Happens | Agent Sees |
|----|---------|-------------|------------|
| **store** | `secret-ops.sh store KEY` | Interactive `read -rsp` → backend | Exit code |
| **inject** | `secret-ops.sh inject KEY --confirm -- cmd args` | Retrieves secret, injects via subshell+exec | Command output |
| **list** | `secret-ops.sh list` | Queries backend for key names | Key names |
| **delete** | `secret-ops.sh delete KEY --confirm` | Removes from backend | Exit code |
| **exists** | `secret-ops.sh exists KEY` | Checks backend | Exit 0=yes, 1=no |

## Backends

| Backend | Tool | OS | Auto-detect Priority |
|---------|------|----|---------------------|
| HashiCorp Vault | `vault` | Any | 1st (if `VAULT_ADDR` set + authenticated) |
| macOS Keychain | `security` | macOS | 2nd |
| Linux keyring | `keyctl` | Linux | 3rd |
| Git Credential Manager | `git credential-manager` | Any | 4th (cross-platform fallback) |

Backend is **pinned on first use** to `~/.config/secret-ops/backend`. No silent downgrade.

## Security Model

- **No reveal** — there is no `get`/`read` command. Only `inject` (scoped subprocess via subshell+exec).
- **Approval gates** — `inject` and `delete` require `--confirm` flag. Agent must ask user before passing it.
- **Fail-closed** — if the pinned backend fails, the operation fails. No fallback chain.
- **No argv leaks** — secrets injected via shell `export` in a subshell, not `env` command argv.
- **Locked backend pinning** — first-use detection uses `flock`/mutex to prevent race conditions.
- **Key validation** — key names restricted to `[A-Za-z0-9_.-]+` (max 256 chars).
- **Audit log** — every operation logged to `~/.config/secret-ops/audit.log` (ops only, never values).
- **AI-Human Principle 3** — agent never asks for, sees, or handles secret values.

## Token Efficiency

SKILL.md is protocol-only (~625 tokens). All logic lives in scripts that are executed but never loaded into context. ~90% reduction vs v0.1.0.

## Authoritative Sources

- [Apple Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
- [Linux keyctl man page](https://man7.org/linux/man-pages/man1/keyctl.1.html)
- [Linux kernel key management](https://www.kernel.org/doc/html/latest/security/keys/core.html)
- [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager)
- [HashiCorp Vault](https://developer.hashicorp.com/vault/docs)
- [AI-Human Interaction Defaults](https://github.com/tyler555g/best-practices/blob/main/packages/content/technology_and_information/data_science_and_ai/ai-human-interaction-defaults.md)
- [12-Factor Agents](https://github.com/humanlayer/12-factor-agents)

## License

MIT
