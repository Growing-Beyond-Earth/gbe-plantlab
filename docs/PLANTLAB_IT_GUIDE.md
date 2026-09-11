# PlantLab Guide for School IT

*Applies to firmware 1.0 and later. Last updated 2026-09-10.*

> **Devices shipped before 2026** run the original MicroPython firmware and
> are not as described in this document until updated. The update is a
> one-time installation over USB, covered by the installation guide
> distributed alongside this document; once applied, the device behaves as
> described here and receives all subsequent updates automatically.

## The program

Growing Beyond Earth (GBE) is a classroom science program created by
Fairchild Tropical Botanic Garden in partnership with NASA. Students grow
edible plants in a standardized growth chamber — the **PlantLab** — and their
experiments contribute to research on growing food in space. The program is
active in more than 500 schools.

## The device

The PlantLab is a benchtop growth chamber controlled by a Raspberry Pi
Pico W microcontroller. It is **not a general-purpose computer**: there is
no operating system, no shell, and no remote login of any kind. The device
runs a single signed firmware image and nothing else.

The device:

- Controls an RGBW LED light panel, a circulation fan, and a general-purpose
  12 V auxiliary output, following a schedule the class writes as a simple
  JSON file (the "program").
- Measures temperature, humidity, CO₂, light, soil moisture, and its own
  electrical power draw. It has **no camera and no microphone**.
- Logs readings to its SD card every 10 minutes and, when connected to
  WiFi, reports the same readings to the GBE server, where the class can
  view current conditions and history online.

Scanning the **QR code on the PlantLab's LED panel** opens that device's
page, which shows its hardware details — MAC address, firmware version,
device ID, and device name — along with the latest version of this and the
other PlantLab documents. This is the quickest way to collect a unit's MAC
address for network registration.

## How a classroom gets set up

The typical flow is entirely through the **GBE Connect** phone app, over
Bluetooth LE, standing next to the device:

1. The teacher plugs in the PlantLab and opens GBE Connect nearby.
2. The app finds the device over Bluetooth and connects.
3. In the app, the teacher names the device and enters the school WiFi
   credentials.
4. From then on the device runs on its own; the app is used for program
   changes, updates, and occasional check-ins.

An SD-card path exists for every configuration task (a settings file placed
on the card), but in practice most teachers use only the app. No step
requires a school-managed computer, browser configuration, or inbound
network access.

**WiFi connectivity is recommended.** A connected PlantLab can be checked
remotely in real time — by the class, from home, or by GBE support during
troubleshooting — and its data reaches the online charts continuously with
no action from the teacher. The device can operate without WiFi, but with a
reduced experience; see "Operation without WiFi" below.

## Network behavior

- WiFi client only: WPA2, 2.4 GHz. The device never creates an access point
  and never listens for inbound connections — there are no open ports to
  scan and no web interface on the device.
- All traffic is outbound:

| Destination | Protocol | Purpose |
|---|---|---|
| `growingbeyond.earth` | TCP 443 (HTTPS) | Telemetry (~every 10 min), program sync, firmware update checks |
| NTP | UDP 123 | Clock synchronization |
| Public web APIs (optional) | TCP 443 (HTTPS) | Only if a classroom program explicitly fetches one (e.g. a weather API) |

- HTTPS certificate verification is enforced against pinned Let's Encrypt
  roots (ISRG Root X1 and X2); a failed certificate check fails the
  connection.
- Programs that fetch public APIs are egress-filtered on the device: requests
  to private and internal address ranges are refused, so a classroom program
  cannot be used to probe the school network.
- WiFi is optional but recommended (see "Operation without WiFi"). Both
  radios can be disabled outright (see "Controls available to IT").

**Recommended placement:** a guest network or IoT VLAN, per normal IoT
practice, with outbound TCP 443 to `growingbeyond.earth` and UDP 123
allowed. No captive portal.

## Bluetooth

Bluetooth LE is the local-setup and check-in channel for GBE Connect.
Relevant design points:

- BLE range is room-scale. The access model is that **physical proximity is
  authorization** — the same model as the SD card slot. There is no PIN or
  pairing step; the device has no display on which to conduct one.
- The one secret that crosses BLE — the WiFi password — does not rely on
  Bluetooth link security. It is encrypted at the application layer (ECDH
  key agreement with AES-GCM) between the app and the device, so it is
  protected regardless of the link.
- What BLE exposes otherwise: live sensor readings, device status, program
  management, and firmware update delivery — the same class of actions
  available to a person standing at the device. The device stores no student
  data for any channel to leak.

## Operation without WiFi

The device operates correctly without WiFi, but the experience is reduced:
there is no remote real-time view of the device, and data reaches the
online charts only when a teacher acts.

- The device keeps its full log on the SD card regardless of connectivity,
  and continues running its program.
- Readings destined for the online charts are cached on the card. If WiFi
  is restored, the backlog uploads automatically.
- When a teacher connects with GBE Connect, the app relays the cached
  backlog (and can deliver firmware updates) through the phone's own
  internet connection. GBE advises teachers of offline units to connect at
  least a few times per week. For network accounting purposes, note that
  data relayed this way reaches `growingbeyond.earth` via the teacher's
  phone rather than the school network.

## What data leaves the school

Telemetry entries contain: device identity (device name, hardware address, firmware
version), sensor readings, light/fan/aux output states, and device health
counters (uptime, memory). No student names or accounts exist on the device,
no personal data is collected, and there is no camera or microphone. WiFi
passwords are never transmitted to GBE — see below.

## Credential handling

- WiFi credentials are stored on internal flash, obfuscated with a
  device-derived key (the RP2040 has no secure element, so this is
  obfuscation rather than hardware-backed encryption — as on most
  microcontroller-class devices).
- They are never written to the SD card and never appear in any log. If
  credentials are supplied via an SD settings file, the file is blanked
  (zero-overwritten, then deleted) immediately after reading.
- Units upgraded from the original MicroPython firmware read that system's
  plaintext `wifi_settings.json` from the card once, then scrub and delete
  it.
- When troubleshooting a device, the GBE team may ask a teacher to send
  `logs/startup/output.txt` from the SD card (boot output, overwritten each
  boot). It contains the firmware version, hardware status, network name,
  and signal strength/scan results — never the WiFi password.

## Firmware updates

- Every firmware image is signed (ECDSA P-256) by the GBE team; signing keys
  are held offline, never on the update server. The device's bootloader
  verifies the signature before an update can run; unsigned or tampered
  images are rejected and the device automatically rolls back to its
  previous firmware.
- Downgrades below a minimum version are refused (rollback protection).
- Delivery: over HTTPS from `growingbeyond.earth` when the device has WiFi,
  or through GBE Connect over Bluetooth when it doesn't. A USB installer
  exists for first-time upgrades and recovery.
- Net effect: firmware cannot be installed by any party other than the GBE
  team, regardless of network position.

## Controls available to IT

- **Network placement** — the device requires nothing from the school LAN;
  guest/IoT segregation involves no loss of functionality. This is the
  recommended way to accommodate the device: it preserves the full
  experience, including remote monitoring.
- **Offline operation** — if WiFi cannot be provided, the device still runs
  its program, logs to the SD card, and remains manageable over Bluetooth,
  with the limitations described under "Operation without WiFi".
- **Radio lockdown** — setting `WIFI_ENABLED` and `BLUETOOTH_ENABLED` to
  `false` in `wifi_bluetooth_enabled.json` on the SD card silences both
  radios entirely. Lockdown can only be set or lifted physically at the
  device — there is no remote or Bluetooth path to change these flags.
  (Re-enabling Bluetooth takes effect at the next power-up.)

## Questions

Fairchild Tropical Botanic Garden — Growing Beyond Earth program:
[gbe@fairchildgarden.org](mailto:gbe@fairchildgarden.org).
