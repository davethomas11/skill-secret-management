# skill-secrets-setup

Interactive Copilot skill for setting up secure secret storage on your workstation.

Detects your OS, available tooling (macOS Keychain, Linux keyring, HashiCorp Vault, Git Credential Manager), scans for plaintext secrets, and configures a resilient `_secret_get` wrapper with JIT fetching.

## Installation

Copy `SKILL.md` into your Copilot skills directory:

```bash
# GitHub Copilot CLI
cp SKILL.md ~/.copilot/skills/secrets-setup/SKILL.md

# Or clone the whole repo
git clone https://github.com/tyler555g/skill-secrets-setup.git ~/.copilot/skills/secrets-setup
```

## Usage

Invoke the skill from any Copilot session:

```
Use the skill tool to invoke the "secrets-setup" skill
```

The skill walks you through an interactive 6-step process:

1. **Detect Environment** — OS, available tools, existing Vault config, plaintext secrets scan
2. **Ask User Context** — Vault access, secret types, persistent env var needs
3. **Recommend Method** — Vault + OS fallback, OS-native only, or Git Credential Manager
4. **Configure** — Install deps, create shell helpers, migrate plaintext secrets
5. **Verify** — Round-trip test, shell function test, Vault connectivity
6. **Educate** — JIT vs export tradeoffs, rotation reminders

## Supported Platforms

| Platform | Backend |
|----------|---------|
| macOS | Keychain (`security` CLI) |
| Linux | Kernel keyring (`keyctl`) |
| Windows | Credential Manager |
| Any | HashiCorp Vault |
| Any | Git Credential Manager |

## License

MIT

## Authoritative Sources

- [Apple Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
- [Linux `keyctl` man page](https://man7.org/linux/man-pages/man1/keyctl.1.html)
- [Linux kernel key management](https://www.kernel.org/doc/html/latest/security/keys/core.html)
- [HashiCorp Vault docs](https://developer.hashicorp.com/vault/docs)
- [HashiCorp Vault Agent](https://developer.hashicorp.com/vault/docs/agent-and-proxy/agent)
- [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager/blob/main/README.md)
- [Windows Credential Manager](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication)
