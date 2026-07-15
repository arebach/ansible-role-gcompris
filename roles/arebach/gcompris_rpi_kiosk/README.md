# gcompris_rpi_kiosk

An Ansible role for deploying a child-safe [GCompris](https://gcompris.net) educational kiosk on Raspberry Pi OS (Debian Bookworm).

## Features

- **Kiosk mode** — auto-launches GCompris on boot via LightDM autologin
- **Age filtering** — restricts activities to a configurable age range (0-9)
- **Network isolation** — 3 modes: `full`, `internal` (LAN only, default), `none` (air-gapped)
- **Touchscreen support** — optional tslib and calibration for official RPi touch display
- **Screen timeout** — configurable DPMS blanking (default 15 min, 0 = disabled)
- **Ctrl+Alt+Backspace exit** — configurable escape hatch (enabled by default)
- **System hardening** — UFW firewall, Bluetooth disabled, automatic OS updates

## Requirements

- Raspberry Pi 4 or 5 running Raspberry Pi OS (Debian Bookworm)
- Ansible 2.15+
- `community.general` collection (for UFW module)
- SSH access to the target Pi (password or key-based)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `gcompris_rpi_kiosk_network_mode` | `internal` | Network access: `full`, `internal`, or `none` |
| `gcompris_rpi_kiosk_mode` | `true` | Enable kiosk mode (auto-launch + autologin) |
| `gcompris_rpi_kiosk_min_age_level` | `0` | Minimum activity difficulty (0-9) |
| `gcompris_rpi_kiosk_max_age_level` | `5` | Maximum activity difficulty (0-9) |
| `gcompris_rpi_kiosk_user` | `kiosk` | Dedicated system user for the kiosk session |
| `gcompris_rpi_kiosk_password` | `kiosk` | Password for kiosk user (change for production) |
| `gcompris_rpi_kiosk_screen_timeout_minutes` | `15` | Screen blank timeout (0 = disabled) |
| `gcompris_rpi_kiosk_block_zap` | `false` | Block Ctrl+Alt+Backspace exit |
| `gcompris_rpi_kiosk_enable_touchscreen` | `false` | Enable tslib touchscreen support |
| `gcompris_rpi_kiosk_update_system` | `true` | Run apt update + dist-upgrade before provisioning |
| `gcompris_rpi_kiosk_reboot` | `true` | Reboot Pi after provisioning |

See [`defaults/main.yml`](defaults/main.yml) for all variables.

## Dependencies

- `community.general` collection (for UFW firewall management)

## Example Playbook

```yaml
- hosts: kiosks
  roles:
    - role: arebach.gcompris_rpi_kiosk.gcompris_rpi_kiosk
      vars:
        gcompris_rpi_kiosk_network_mode: internal
        gcompris_rpi_kiosk_min_age_level: 0
        gcompris_rpi_kiosk_max_age_level: 5
```

## Security

- Default network mode is `internal` (LAN only, internet blocked)
- UFW firewall with default deny + explicit LAN allow rules
- Bluetooth disabled
- Default kiosk password is `kiosk` — **change it for production deployments**
- Use `ansible-vault` to encrypt inventory passwords

## License

MIT

## Author

Ari Rebach ([@arebach](https://github.com/arebach))
