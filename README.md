# GCompris RPi Kiosk Ansible Role
=================================

[![CI/CD](https://github.com/arebach/ansible-role-gcompris/actions/workflows/ci-cd.yml/badge.svg)](https://github.com/arebach/ansible-role-gcompris/actions/workflows/ci-cd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-arebach.gcompris__rpi__kiosk-blue.svg)](https://galaxy.ansible.com/arebach/gcompris_rpi_kiosk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![ansible-lint: production](https://img.shields.io/badge/ansible--lint-production-green.svg)](https://ansible-lint.com/)
[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)

**Ansible Galaxy role for deploying a child-safe, locked-down GCompris kiosk on Raspberry Pi.**

`ansible-galaxy install arebach.gcompris_rpi_kiosk`

## Overview

This role provisions a Raspberry Pi as a **dedicated educational kiosk** running [GCompris](https://gcompris.net/), the free educational software suite for children aged 2–10. It locks the system into a single-purpose, tamper-resistant kiosk session with configurable network restrictions and age-appropriate activity filtering.

### Key Features

- **Full kiosk lockdown** — blocks VT switching, X server termination, shell access
- **3 network modes** — full access, internal-only (recommended), or air-gapped
- **Age-level filtering** — restrict activities to specific developmental stages
- **Autologin + auto-start** — boots directly into GCompris, no login screen
- **Auto-reboot** — optionally reboots the Pi after provisioning so all changes take effect
- **UFW safety flush** — clears stale firewall rules before installing packages (prevents apt lockup from previous network restrictions)
- **Configurable screen timeout** — display blanks after N minutes of inactivity (default 15, or 0 for always-on)
- **Touchscreen ready** — optional tslib calibration for capacitive touch displays
- **Diagnostics playbook** — 11-check health check for the entire boot-to-launch chain
- **Molecule-tested** — CI/CD validates configuration files in Docker containers (static config verification, not runtime)
- **Galaxy-compatible** — follows `arebach` namespace, Debian Bookworm target

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install arebach.gcompris_rpi_kiosk
```

### From This Repository

```bash
git clone https://github.com/arebach/ansible-role-gcompris.git
cd ansible-role-gcompris
```

## Usage

### Quick Start

```bash
# 1. Edit inventory with your Pi's IP address and credentials
#    inventory.yml → change ansible_host, ansible_user,
#    ansible_password, and ansible_become_pass

# 2. (Optional) Set up SSH key auth instead of password:
#    ssh-copy-id pi@192.168.1.100
#    (then remove ansible_password from inventory)

# 3. Run the playbook (Pi will auto-reboot at the end)
ansible-playbook -i inventory.yml playbooks/provision.yml

# 4. After reboot, the Pi auto-logs in as the kiosk user
#    and launches GCompris in fullscreen kiosk mode.
#    Manual login credentials (if autologin fails):
#      Username: kiosk
#      Password: kiosk
```

### Customization Examples

**Air-gapped kiosk for unsupervised use:**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_rpi_kiosk_network_mode=none \
  -e gcompris_rpi_kiosk_mode=true
```

**Internal-only with age filtering (classroom, ages 4–8):**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_rpi_kiosk_network_mode=internal \
  -e gcompris_rpi_kiosk_min_age_level=2 \
  -e gcompris_rpi_kiosk_max_age_level=7
```

**Normal desktop with GCompris (not kiosk):**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_rpi_kiosk_mode=false
```

**Skip OS update (offline/air-gapped reprovisioning):**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_rpi_kiosk_update_system=false
```

**With touchscreen support:**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_rpi_kiosk_enable_touchscreen=yes
```

**Allow Ctrl+Alt+Backspace as maintenance exit:**
```bash
# Default behavior — Ctrl+Alt+Backspace exits to greeter
ansible-playbook -i inventory.yml playbooks/provision.yml

# Lock down completely — block Ctrl+Alt+Backspace
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_rpi_kiosk_block_zap=true
```
By default, pressing `Ctrl+Alt+Backspace` kills the X server and returns to the LightDM greeter for maintenance login. GCompris auto-restarts on next login. Set `block_zap=true` for unattended deployments.

### Diagnostics

If GCompris doesn't auto-start after reboot, run the diagnostics playbook:

```bash
ansible-playbook -i inventory.yml playbooks/diagnostics.yml
```

This checks all links in the boot-to-launch chain and reports ✅ PASS or ❌ FAIL for each:

1. LightDM enabled on boot
2. LightDM service currently running
3. Default display manager set to LightDM
4. Autologin config references `gcompris-kiosk` session
5. No conflicting autologin in main `lightdm.conf`
6. `.desktop` entry has `Type=XSession` (and no unsupported session types)
7. Session wrapper script exists and is executable
8. `gcompris-qt` binary installed in PATH
9. Display server is X11 (not Wayland)
10. Kiosk user exists with correct device groups
11. LightDM logs (last 30 lines from current boot)
12. Active login sessions

## Variables Reference

### Network Modes (`gcompris_rpi_kiosk_network_mode`)

| Value | Behavior | Use Case |
|-------|----------|----------|
| `"full"` | All network services active | Supervised use, A/V downloads needed |
| `"internal"` | LAN only, internet blocked (default) | Classroom, library — safe default |
| `"none"` | Air-gapped, no network | Public/unattended, maximum safety |

### Kiosk Mode (`gcompris_rpi_kiosk_mode`)

| Value | Behavior |
|-------|----------|
| `true` (default) | Full lockdown, autologin, auto-start GCompris |
| `no` | Normal desktop, GCompris as regular app |

### Age Filtering (`gcompris_rpi_kiosk_min_age_level`, `gcompris_rpi_kiosk_max_age_level`)

GCompris has 10 age levels (0–9). Activities are filtered to the specified range:

| Level | Age Range | Example Activities |
|-------|-----------|--------------------|
| 0 | 2–3 | Colors, sounds, simple puzzles |
| 1–2 | 3–5 | Reading, math basics, geography |
| 3–4 | 5–7 | Science, programming games |
| 5–6 | 7–9 | Algebra, geography, drawing |
| 7–9 | 9–12+ | Advanced science, programming |

Set both to the same value for a tight range (e.g., `min: 2 max: 2` for ages 3–5 only).

### Display & Input

- **`gcompris_rpi_kiosk_force_x11`** — `true` — Force X11 compositor (required for kiosk)
- **`gcompris_rpi_kiosk_force_hdmi`** — `false` — Force HDMI output even if undetected
- **`gcompris_rpi_kiosk_enable_touchscreen`** — `false` — Enable tslib for capacitive touch
- **`gcompris_rpi_kiosk_screen_timeout_minutes`** — `15` — Screen blank timeout in minutes (0 = always on)
- **`gcompris_rpi_kiosk_block_zap`** — `false` — Block `Ctrl+Alt+Backspace` (X server kill). Set to `true` for full lockdown. When `false` (default), `Ctrl+Alt+Backspace` kills GCompris and returns to the LightDM greeter for maintenance

### System Configuration

- **`gcompris_rpi_kiosk_user`** — `"kiosk"` — Dedicated system user for the kiosk session
- **`gcompris_rpi_kiosk_group`** — `"kiosk"` — System group for the kiosk user
- **`gcompris_rpi_kiosk_password`** — `"kiosk"` — Password for the kiosk user (used for manual login if autologin fails; change for production)
- **`gcompris_rpi_kiosk_autologin_timeout`** — `0` — LightDM autologin delay in seconds (0 = immediate)
- **`gcompris_rpi_kiosk_local_pkg_dir`** — `/var/cache/gcompris_rpi_kiosk_pkg` — Offline package staging directory
- **`gcompris_rpi_kiosk_reboot`** — `true` — Reboot the Pi after provisioning so all changes take effect (set to `false` for CI/CD)
- **`gcompris_rpi_kiosk_update_system`** — `true` — Run `apt-get update && dist-upgrade` as the first action before any provisioning. Ensures the host OS is fully patched. Disable with `false` (e.g. for offline/air-gapped reprovisioning)

## Network Modes in Detail

### Full Mode
No restrictions. GCompris can download A/V content packs, sync cloud saves, and use all network features.

### Internal-Only Mode (Recommended Default)
UFW firewall with default deny on both incoming and outgoing. Explicit allow rules for RFC1918 private ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) on ports 53 (DNS), 67 (DHCP), 22 (SSH), 80/443 (HTTP/HTTPS), 3389 (RDP). **Internet is blocked** — children cannot browse the web or access external servers. LAN games and local media sharing still work. Bluetooth is disabled.

### None (Air-Gapped) Mode
All network services are stopped and disabled: Wi-Fi, Ethernet (via NetworkManager/dhcpcd), Bluetooth, and ModemManager. The device **cannot reach any network** — not even a local LAN. This is the maximum security posture.

## Project Structure

```
ansible-role-gcompris/
├── galaxy.yml                    # Galaxy metadata for publishing
├── CHANGELOG.md                 # Version history
├── CONTRIBUTING.md              # Contributor guidelines
├── CODE_OF_CONDUCT.md           # Community code of conduct
├── SECURITY.md                  # Security policy and reporting
├── .github/
│   ├── workflows/ci-cd.yml       # Lint → Molecule → Galaxy pipeline
│   ├── ISSUE_TEMPLATE/           # Bug report, feature request, support templates
│   ├── pull_request_template.md  # PR checklist template
│   ├── dependabot.yml            # Automated GitHub Actions dependency updates
│   ├── stale.yml                 # Auto-close stale issues/PRs
│   └── FUNDING.yml               # Sponsorship configuration
├── playbooks/
│   ├── provision.yml             # Example deployment playbook
│   └── diagnostics.yml           # Boot-to-launch chain health check
├── inventory.yml                 # Example host inventory
├── vars/deployment.yml           # Override examples
├── docs/
│   └── gcompris-configuration.md # GCompris CLI flag reference
├── meta/runtime.yml              # Galaxy runtime requirements
├── roles/arebach/gcompris_rpi_kiosk/
│   ├── defaults/main.yml         # Overridable defaults (documented above)
│   ├── vars/main.yml             # Internal variables, package lists
│   ├── tasks/main.yml            # 20 provisioning tasks + pre-tasks
│   ├── tasks/network-full.yml    # Network mode: full
│   ├── tasks/network-internal.yml # Network mode: internal
│   ├── tasks/network-none.yml    # Network mode: none (air-gapped)
│   ├── handlers/main.yml         # Service restarts (LightDM, logind, DPMS)
│   ├── templates/
│   │   ├── 10-autologin.conf.j2       # LightDM autologin config
│   │   ├── 10-kiosk.conf.j2           # X11 security flags (DontVTSwitch, conditional DontZap)
│   │   ├── gcompris-kiosk-session.j2  # Kiosk wrapper script
│   │   ├── gcompris-kiosk.desktop.j2  # Session .desktop file (Type=XSession)
│   │   └── 99-screen-timeout.conf.j2  # Xorg DPMS / screen blanking config
│   ├── packages/                 # Offline .deb staging directory
│   ├── meta/main.yml             # Galaxy metadata
│   ├── meta/requirements.yml     # Collection dependencies (community.general)
│   ├── meta/runtime.yml           # Role runtime requirements
│   └── molecule/default/         # Molecule test suite (Docker-based)
├── .gitignore
└── README.md
```

## CI/CD

The GitHub Actions pipeline runs on every push/PR:

1. **Lint** — `ansible-lint` on the role
2. **Test** — Molecule suite (Debian 12 Docker container)
3. **Deploy** — Publish to Ansible Galaxy (on push to `main`)

## Security Considerations

- **Kiosk lockdown** is the first layer — blocks VT switching, X kill, shell access
- **Network restriction** is the second layer — prevents any network-based attacks
- **Age filtering** is the third layer — ensures age-appropriate content
- **Air-gap mode** is the fourth layer — complete physical network isolation
- All four layers work together; none alone is sufficient for unattended deployment
- **Kiosk user password** — default is `"kiosk"`; change via `gcompris_rpi_kiosk_password` for production
- **Inventory passwords** — the example `inventory.yml` contains placeholder values for both `ansible_password` (SSH login) and `ansible_become_pass` (sudo). For real deployments, use `ansible-vault` to encrypt the inventory file or configure SSH key-based authentication via `ssh-copy-id` and remove both password fields

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for development setup, coding standards, testing requirements, and the pull request process.

By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md).

## Security

Found a vulnerability? Please see [SECURITY.md](SECURITY.md) for responsible disclosure. **Do not open a public issue for security vulnerabilities.**

## License

MIT — see [LICENSE](LICENSE) for details.

## Author

**Ari Rebach** · Arebach Consulting · [GitHub](https://github.com/arebach)

## Related

- [Sugar RPi Kiosk Role](https://github.com/arebach/ansible-role-sugar) — Sugar OS educational kiosk (same author, same pattern)
