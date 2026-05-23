# BentoClaw Buddy — Firmware Releases

**Hardware**: KIT_PSE84_AI (PSoC Edge E84 + AIROC CYW55500 DualBand combo radio)
**Brand**: **BENTO : : Make Anything.**

This repository hosts the **flashable firmware images** for BentoClaw — a MicroPython runtime on PSoC Edge E84 that bridges hardware sensors + radios to a desktop companion app over BLE.

> ✨ This is a binary-release-only repository.

---

## What is Bento Buddy?

Bento Buddy is the on-device service that turns a Bento AI Kit board into a **BLE peripheral your laptop can talk to**. The desktop companion (separate project) scans, pairs, and exchanges JSON commands over the standard Nordic UART Service (NUS) — so your Mac, Linux, or Windows machine can read sensors, drive GPIO, query Wi-Fi state, or run MicroPython snippets on the board without any cable.

Built into the same firmware:

- **DualBand concurrency** — Wi-Fi 2.4/5 GHz and Bluetooth Low Energy run side-by-side on a single CYW55500 radio. No mode-switching, no manual coordination.
- **MicroPython REPL** — full Python prompt over USB-UART (115200 baud) and over BLE NUS. Import `wifi`, `mqtt`, `sensors`, `gpio` and script the board live.
- **Lazy BLE bring-up** — BLE NUS stack initializes on first request and rearms advertising on demand, so a board you never pair never pays the heap cost.
- **LE Secure Connections + bonding** — passkey pairing with DisplayOnly IO capability; unbonded peers are rejected at the GATT layer.
- **Auto-reconnect Wi-Fi** — credentials persist to internal flash (LittleFS `/​.wifi_creds`) and reconnect automatically across reboots.
- **Sensor dashboard** — live readouts for the AI Kit's on-board IMU (BMI270), magnetometer (BMM350), barometer (DPS368), and humidity sensor (SHT40).
- **Plain MQTT client** — connect to any MQTT broker on your LAN via the built-in `mqtt` MicroPython module.

---

## What's in each release

Every release attaches the following assets:

| File | Purpose |
|---|---|
| `bento-buddy-vX.Y.Z-KIT_PSE84_AI.hex` | **Single-shot programmable image** — flash this one file via the **Bento Desktop Buddy** application to bring up all three cores at once |
| `SHA256SUMS.txt` | Integrity hashes for the `.hex` image |

The version tag (`vX.Y.Z`) corresponds to `BENTO_BUDDY_FW_VERSION` reported by `bento.info` over BLE NUS and by the firmware boot banner.

---

## Talk to the board

### Over USB serial
```bash
# macOS / Linux
screen /dev/cu.usbmodem* 115200
# or
picocom -b 115200 /dev/ttyACM0
```

You'll get a MicroPython REPL:
```python
>>> import wifi
>>> wifi.connect("YourSSID", "YourPassword")
[WiFi] Connecting to 'YourSSID'...
[WiFi] Connected! IP=192.168.1.x
>>> import sensors
>>> sensors.imu()   # BMI270 accel + gyro
>>> sensors.env()   # DPS368 + SHT40
```

### Over BLE
The board advertises as `Bento-XXXX` (XXXX = last 4 hex of MAC). Pair from the **Bento Desktop Buddy** app — sensor data and commands stream over Nordic UART Service.

---

## Verified capabilities (independent V&V, 2026-05)

| Surface | Status |
|---|---|
| BLE NUS pair + bond (LE Secure Connections, passkey) | ✅ Validated |
| Wi-Fi DualBand (2.4 + 5 GHz) connect / scan / reconnect | ✅ Validated |
| MicroPython REPL over USB-UART + BLE NUS | ✅ Validated |
| **Plain MQTT** client → arbitrary broker on the LAN (publish + subscribe) | ✅ End-to-end validated against local `eclipse-mosquitto` |
| Wi-Fi + BLE concurrent (no radio handoff) | ✅ Validated |
| Sensor dashboard (IMU + mag + baro + humidity) | ✅ Validated |
| TLS / mTLS MQTT | ⚠️ Requires OPTIGA Trust M secure-element add-on (the DualBand variant's RAM budget cannot fit a software-only TLS handshake alongside concurrent Wi-Fi + BLE + MicroPython) |

---

## Versioning

Tags follow `vMAJOR.MINOR.PATCH` matching `BENTO_BUDDY_FW_VERSION` in the firmware. Each release tag points at the exact firmware build that emitted the assets — verify with:

```python
>>> import bento
>>> bento.info()
{"fw":"1.1.0","board":"BENTO KIT_PSE84_AI","tools":30,"proto":2}
```

---

## Companion software

- **Bento Desktop Buddy** — the host-side BLE bridge + LLM chat panel (macOS / Linux / Windows). Use it to scan, pair, send NUS commands, and chat to the board through your preferred LLM provider (BYOK).

---

## Support & links

- Hardware vendor docs: [Infineon PSoC Edge E84 product page](https://www.infineon.com/cms/en/product/microcontroller/32-bit-psoc-arm-cortex-microcontroller/psoc-edge-e8-series/)
- Combo radio: [AIROC CYW55500](https://www.infineon.com/cms/en/product/wireless-connectivity/airoc-wi-fi-plus-bluetooth-combos/wi-fi-6-6e/cyw55500/)
- For issues with these binary releases, open a GitHub issue on this repository
- For Bento Buddy desktop app, see its companion repository
