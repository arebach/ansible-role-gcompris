# Changelog

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
