# Docan Panda KS BMS → ESPHome → Home Assistant

A write-up of integrating a **Docan Panda 51.2V battery with a KS-style BMS** into **Home Assistant** using an **ESP32 Bluetooth proxy** and the excellent [`syssi/esphome-ks-bms`](https://github.com/syssi/esphome-ks-bms) component.

## What this solves

If your battery has:
- a touchscreen or front panel with its own Bluetooth identity
- a mobile app that connects fine
- a KS-style BMS underneath
- and an ESP32 that refuses to find or connect to it

...you may be targeting the **wrong Bluetooth radio**.

That was the issue here.

## Root cause

The battery touchscreen Bluetooth MAC was **not** the same as the **actual BMS BLE module**.

The real BMS advertised separately and needed to be identified by:
- its advertised BLE name
- service UUID `FF00`
- characteristics `FF01` and `FF02`

Using the touchscreen Bluetooth MAC in ESPHome caused discovery/connection failure.

## Working approach

1. Put an ESP32 physically near the battery.
2. Flash a **scan-only** ESPHome config.
3. Use `esp32_ble_tracker` with active scanning.
4. Watch logs for a device that matches the BMS name and exposes service `FF00`.
5. Use **that** MAC in `ble_client`.
6. Add the KS BMS external component.
7. Expose entities to Home Assistant.

## Repo contents

- `esphome/ble-scan-example.yaml` — sanitized scan-only config for finding the real BMS BLE endpoint
- `esphome/ks-bms-example.yaml` — sanitized working example using `syssi/esphome-ks-bms`
- `home-assistant/template-sensors.yaml` — example HA template ideas for usable battery metrics

## Hardware used

- Docan Panda 51.2V battery pack with KS-style BMS
- ESP32 running ESPHome
- Home Assistant

## Important notes

### 1) The touchscreen may lie to you
Not maliciously. Just architecturally 😏

Some battery systems expose a Bluetooth radio for the display/tablet/touchscreen layer, while the BMS telemetry comes from a **different BLE module**.

If you use the obvious MAC and ESPHome sees nothing useful, scan first instead of assuming BLE is broken.

### 2) Wi-Fi mistakes can look like BLE problems
During troubleshooting, a bad Wi-Fi target or moving the ESP to a different area can make the device vanish and waste an hour that should have been spent blaming Bluetooth.

Make sure the ESP32 stays online when moved near the battery.

### 3) Keep BLE enabled after reboot
Using:

```yaml
restore_mode: RESTORE_DEFAULT_ON
```

for the BLE client switch helps the node reconnect automatically after restart.

## Sanitization policy used for this repo

Removed before publishing:
- local IPs
- SSIDs
- API keys
- OTA passwords
- internal hostnames
- local BLE scan noise from unrelated personal devices
- any site-specific Home Assistant entity names that reveal private infrastructure

## General public takeaway

If a KS BMS battery app works but ESPHome does not, do **not** assume the app-visible Bluetooth identity is the real BMS endpoint.

Scan for the actual BLE advertisement and identify it by:
- advertised name
- service UUIDs
- characteristic UUIDs

That’s the trick.

## Credit

Huge credit to:
- [`syssi/esphome-ks-bms`](https://github.com/syssi/esphome-ks-bms)
- the ESPHome ecosystem
- everyone who documents weird battery/BLE behavior instead of suffering in silence

## License

MIT
