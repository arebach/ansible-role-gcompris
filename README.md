# GCompris RPi Kiosk Ansible Role
=================================

**Ansible Galaxy role for deploying a child-safe, locked-down GCompris kiosk on Raspberry Pi.**

`ansible-galaxy install arebach.gcompris_rpi_kiosk`

## Overview

This role provisions a Raspberry Pi as a **dedicated educational kiosk** running [GCompris](https://gcompris.net/), the free educational software suite for children aged 2–10. It locks the system into a single-purpose, tamper-resistant kiosk session with configurable network restrictions and age-appropriate activity filtering.

### Key Features

- **Full kiosk lockdown** — blocks VT switching, X server termination, shell access
- **3 network modes** — full access, internal-only (recommended), or air-gapped
- **Age-level filtering** — restrict activities to specific developmental stages
- **Autologin + auto-start** — boots directly into GCompris, no login screen
- **Screensaver disabled** — display stays always-on for unattended use
- **Touchscreen ready** — optional tslib calibration for capacitive touch displays
- **Molecule-tested** — CI/CD validates configuration in Docker containers
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
# 1. Edit inventory with your Pi's IP
#    inventory.yml → change ansible_host to your Pi's address

# 2. Run the playbook
ansible-playbook -i inventory.yml playbooks/provision.yml
```

### Customization Examples

**Air-gapped kiosk for unsupervised use:**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_network_mode=none \
  -e gcompris_kiosk_mode=yes
```

**Internal-only with age filtering (classroom, ages 4–8):**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_network_mode=internal \
  -e gcompris_min_age_level=2 \
  -e gcompris_max_age_level=7
```

**Normal desktop with GCompris (not kiosk):**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_kiosk_mode=no
```

**With touchscreen support:**
```bash
ansible-playbook -i inventory.yml playbooks/provision.yml \
  -e gcompris_enable_touchscreen=yes
```

## Variables Reference

### Network Modes (`gcompris_network_mode`)

| Value | Behavior | Use Case |
|-------|----------|----------|
| `"full"` | All network services active | Supervised use, A/V downloads needed |
| `"internal"` | LAN only, internet blocked (default) | Classroom, library — safe default |
| `"none"` | Air-gapped, no network | Public/unattended, maximum safety |

### Kiosk Mode (`gcompris_kiosk_mode`)

| Value | Behavior |
|-------|----------|
| `yes` (default) | Full lockdown, autologin, auto-start GCompris |
| `no` | Normal desktop, GCompris as regular app |

### Age Filtering (`gcompris_min_age_level`, `gcompris_max_age_level`)

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

| Variable | Default | Description |
|----------|---------|-------------|
| `gcompris_force_x11` | `true` | Force X11 compositor (required for kiosk) |
| `gcompris_force_hdmi` | `false` | Force HDMI output even if undetected |
| `gcompris_enable_touchscreen` | `false` | Enable tslib for capacitive touch |
| `gcompris_screen_timeout_minutes` | `15` | Screen blank timeout in minutes (0 = always on) |
| `gcompris_block_exit_keys` | `true` | Block VT switch / X kill combinations |

## Network Modes in Detail

### Full Mode
No restrictions. GCompris can download A/V content packs, sync cloud saves, and use all network features.

### Internal-Only Mode (Recommended Default)
Firewall rules allow traffic only to RFC1918 private ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) on ports 53 (DNS), 67/68 (DHCP), 22 (SSH), 80/443 (HTTP/HTTPS), 3389 (RDP). **Internet is blocked** — children cannot browse the web or access external servers. LAN games and local media sharing still work. Bluetooth is disabled.

### None (Air-Gapped) Mode
All network services are stopped and disabled: Wi-Fi, Ethernet (via NetworkManager/dhcpcd), Bluetooth, and ModemManager. The device **cannot reach any network** — not even a local LAN. This is the maximum security posture.

## Project Structure

```
ansible-role-gcompris/
├── galaxy.yml                    # Galaxy metadata for publishing
├── .github/workflows/ci-cd.yml   # Lint → Molecule → Galaxy pipeline
├── playbooks/provision.yml       # Example deployment playbook
├── inventory.yml                 # Example host inventory
├── vars/deployment.yml           # Override examples
├── roles/arebach/gcompris_rpi_kiosk/
│   ├── defaults/main.yml         # Overridable defaults (documented above)
│   ├── vars/main.yml             # Internal variables, package lists
│   ├── tasks/main.yml            # 19 provisioning tasks
│   ├── tasks/network-full.yml    # Network mode: full
│   ├── tasks/network-internal.yml # Network mode: internal
│   ├── tasks/network-none.yml    # Network mode: none (air-gapped)
│   ├── handlers/main.yml         # Service restarts (LightDM, logind, UFW)
│   ├── templates/
│   │   ├── 10-autologin.conf.j2  # LightDM autologin config
│   │   ├── gcompris-kiosk-session.j2  # Kiosk wrapper script
│   │   └── gcompris-kiosk.desktop.j2    # Session .desktop file
│   ├── files/
│   │   └── 10-kiosk.conf         # X11 server flags (DontVTSwitch, DontZap)
│   ├── meta/main.yml             # Galaxy metadata
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
- All three layers work together; none alone is sufficient for unattended deployment

## License

MIT — see [LICENSE](LICENSE) for details.

## Author

**Ari Rebach** · Arebach Consulting · [GitHub](https://github.com/arebach)

## Related

- [Sugar RPi Kiosk Role](https://github.com/arebach/ansible-role-sugar) — Sugar OS educational kiosk (same author, same pattern)
