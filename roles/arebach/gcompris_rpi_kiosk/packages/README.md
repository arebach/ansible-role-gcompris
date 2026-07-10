# GCompris RPi Kiosk — Local Package Staging
# =============================================
# If you need to deploy this role on a Pi **without internet access**,
# pre-download all .deb dependencies on a connected machine and copy
# them here before running the playbook.

## How to pre-download packages

On a Debian/Ubuntu machine (or the Raspberry Pi before it goes air-gapped):

```bash
# Install apt-offline or use apt-get --download-only
sudo apt-get update
sudo apt-get install --download-only gcompris-qt lightdm tslib x11-utils x11-xserver-utils

# Copy all downloaded .deb files:
cp /var/cache/apt/archives/*.deb /path/to/this-directory/
```

## Files to include

At minimum, include these packages:

- `gcompris-qt` — the educational software (Qt version)
- `lightdm` — display manager for autologin
- `tslib` — touchscreen calibration support
- `x11-utils` + `x11-xserver-utils` — X11 tools (includes xset for DPMS control)
- `ufw` + `iptables` — firewall (for internal/none network modes)
- `libraspberrypi-bin` — HDMI configuration (provides `tvservice` utility)

## After copying

Place the `.deb` files in this `packages/` directory. The role will
install from local files instead of the remote repository.
