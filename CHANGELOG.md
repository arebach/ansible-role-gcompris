# Changelog

## 1.2.0

- **Feature: configurable Ctrl+Alt+Backspace exit** — new `gcompris_rpi_kiosk_block_zap` variable (default: `true`); set to `false` to allow `Ctrl+Alt+Backspace` to kill the X server and return to the LightDM greeter for maintenance
- Converted `10-kiosk.conf` from static file to Jinja2 template (`10-kiosk.conf.j2`) with conditional `DontZap` based on the new variable
- `DontVTSwitch` remains always-on (no reason to allow VT switching in kiosk mode)
- Removed empty `files/` directory
- Updated molecule verify.yml and README documentation

## 1.1.0

- **Fix: kiosk autostart** — `.desktop` entry changed from `Type=Application` to `Type=XSession` (required by LightDM for session entries)
- **Fix: unsupported session type** — removed `X-LightDM-Session-Type=xlocal` (not supported by LightDM on Raspberry Pi OS Bookworm)
- **Fix: LightDM autologin override** — strips conflicting `autologin-user` lines from main `/etc/lightdm/lightdm.conf` that Raspberry Pi OS bakes in by default
- **Fix: kiosk user missing groups** — added `video,input,tty,audio,plugdev` for display/input device access
- **Fix: kiosk user missing password** — added `gcompris_rpi_kiosk_password` variable (default: `kiosk`) with SHA-512 hashing
- **Fix: gcompris-qt install silently failing** — removed `failed_when: false` from install task so errors surface
- **Fix: UFW blocking apt** — added pre-task to flush existing UFW rules before package installation
- **Fix: LightDM not default DM** — added task to set `/etc/X11/default-display-manager`
- **Fix: LightDM not enabled on boot** — added systemd task to ensure `lightdm.service` is enabled
- **Feature: auto-reboot** — new `gcompris_rpi_kiosk_reboot` variable (default: `true`) reboots Pi after provisioning
- **Feature: diagnostics playbook** — 12-check health check for the entire boot-to-launch chain
- **Galaxy: metadata** — added `linux` tag, `repository` key, `CHANGELOG.md`, `meta/runtime.yml` (requires_ansible >=2.15.0)
- ansible-lint production profile clean

## 1.0.0

- Initial release of the GCompris RPi Kiosk role
- Fullscreen kiosk mode with GCompris-Qt CLI flags
- Three network modes: full, internal (default), none
- Variable screen timeout via Xorg DPMS
- X11 security lockdown (DontZap, DontVTSwitch, getty masking)
- LightDM autologin with custom XSession
- Touchscreen support via tslib
- Age-level activity filtering (difficulty 1-6)
- ansible-lint production profile clean
