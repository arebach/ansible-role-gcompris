# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| 1.2.x   | ✅ Yes    |
| < 1.2   | ❌ No     |

## Reporting a Vulnerability

If you discover a security vulnerability in this role, please report it responsibly.

**Do NOT open a public GitHub issue for security vulnerabilities.**

Instead, email **ari@arebach.com** with:

1. A description of the vulnerability
2. Steps to reproduce
3. Potential impact
4. Suggested fix (if any)

You will receive a response within 48 hours. If the vulnerability is confirmed, a patch will be released and credited (unless you prefer to remain anonymous).

## Security Features

This role implements multiple security layers for kiosk deployments:

- **Kiosk lockdown** — blocks VT switching (`DontVTSwitch`), X server termination (`DontZap`, configurable), getty masking
- **Network restriction** — UFW firewall with RFC1918 allowlist (internal mode) or full air-gapping (none mode)
- **User isolation** — dedicated `kiosk` user with minimal groups, no sudo access
- **Session lockdown** — GCompris `--enable-kioskmode` hides Quit/Config buttons
- **Age filtering** — GCompris `--difficulty` flag restricts activities by developmental level

## Security Considerations for Deployers

- **Default password:** The kiosk user has password `kiosk` by default. Change via `gcompris_rpi_kiosk_password` for any real deployment.
- **UFW rules:** Network restrictions are applied at the end of the role. A pre-task flushes existing rules before installation to prevent apt lockup.
- **LightDM autologin:** The role strips conflicting autologin entries from the OS-installed `lightdm.conf` and sets its own in `lightdm.conf.d/`.
- **Ctrl+Alt+Backspace:** Enabled by default (`gcompris_rpi_kiosk_block_zap: false`) for maintenance access. Set to `true` for unattended deployments.
- **SSH access:** Not modified by this role. Configure SSH separately if needed for remote management.
