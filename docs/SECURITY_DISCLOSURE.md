# GBE-Pico — Network & Security Notes for School IT

*Applies to firmware 0.8.x and later. Last updated 2026-07-26.*

## Network footprint

- WPA2 client, 2.4 GHz. Outbound connections only — no AP mode, no listening
  services.
- Outbound: HTTPS to `growingbeyond.earth` (telemetry ~every 10 min, firmware
  updates), NTP, and any public web APIs used in classroom programs.
- Bluetooth LE for local setup via the companion app.
- Works fully offline; WiFi is optional and can be disabled.

## WiFi credential handling

- Stored on internal flash, which can only be accessed with developer tools. 
  Contents are obfuscated with a device-derived key (the RP2040 has no secure 
  key storage, so this is obfuscation, not encryption — as on most 
  microcontroller-class devices).
- Never written to the SD card and never printed to any log. The SD setup
  file (`set_wifi_credentials.json`) is blanked immediately after reading.
- Devices upgraded from the MicroPython firmware read that system's plaintext
  `wifi_settings.json` once, then delete it from the card.
- SD files that held a password are zero-overwritten before deletion.

## Firmware updates

- Delivered over HTTPS from `growingbeyond.earth`, or via the companion app
  over Bluetooth.
- Every image is cryptographically signed (ECDSA P-256) by the GBE team;
  signing keys are held offline and never on the update server.
- The device's bootloader verifies the signature before any update runs.
  Unsigned or tampered images are rejected and the device rolls back to its
  previous firmware automatically.
- Downgrades below a minimum version are refused (rollback protection).
- Net effect: firmware cannot be installed by anyone but the GBE team.

## Recommendations

- Use a guest network or IoT VLAN, per normal IoT practice.
- Allow outbound TCP 443 to `growingbeyond.earth` and UDP 123 (NTP). No
  captive portal.

## Access model

Physical access = authorized to configure (SD card, USB console, nearby
Bluetooth pending the PIN update). The device stores no student data.

**Radio lockdown:** setting `WIFI_ENABLED` and `BLUETOOTH_ENABLED` to `false`
in `wifi_bluetooth_enabled.json` on the SD card silences both radios; the
device then runs fully offline and is configurable only via the card or USB
console. Lockdown can only be set or lifted physically — there is no remote
or Bluetooth path to change these flags. (Re-enabling Bluetooth takes effect
at the next power-up.)

## Support diagnostics

When asked to troubleshoot a device, the GBE team may ask a teacher to send 
`logs/startup/output.txt` from the SD card (boot output, overwritten each 
boot): firmware version, hardware status, network name, signal strength/scan 
results. Never the WiFi password.

Fairchild Tropical Botanic Garden — Growing Beyond Earth program.
