# Pala One 2.0 — Vision Master E213 port

Port of the [Pala One 2.0 e-reader firmware](https://www.youtube.com/@palaone) (project 024) to the **Heltec Vision Master E213 (HT-VME213)** board.

The original firmware targets the Heltec Wireless Paper V1.1 and V1.2 boards. This port uses the V1.1 sketch as the base (it already drives the same `LCMEN2R13EFC1` panel controller IC that the E213 uses) and changes the minimum needed to retarget the new hardware.

## What changed vs. the upstream V1.1 sketch

1. `EInkDisplay_WirelessPaperV1_1` → `EInkDisplay_VisionMasterE213` (3 references).
2. `HAS_BATTERY` set to `0` for the first build. The E213 uses `GPIO 17` for `ADC_CTRL` (the V1.x boards used `GPIO 19`); the VBAT ADC pin is not documented in the public datasheet and must be confirmed from the schematic before re-enabling battery reading.
3. `partitions.csv` rewritten for the E213's **16 MB** flash (the V1.x boards have 8 MB).

All UI, library, bookmarks, WiFi upload, and sleep logic is untouched.

## Build environment

Required Arduino IDE setup (tested with Arduino IDE 2.x):

| Setting | Value |
|---|---|
| **Boards Manager URL** | `https://espressif.github.io/arduino-esp32/package_esp32_index.json` |
| **Board** | `Heltec Vision Master E213` (under "ESP32 Arduino"; if missing, fall back to `ESP32S3 Dev Module`) |
| **Flash Size** | `16 MB (128Mb)` |
| **Partition Scheme** | `Custom` (uses `partitions.csv` from this folder) |
| **USB CDC On Boot** | `Enabled` |
| **PSRAM** | `OPI PSRAM` |
| **CPU Frequency** | `240 MHz` |
| **Upload Speed** | `921600` |

If you have to fall back to "ESP32S3 Dev Module", also add this build flag (Sketch → Build Properties is awkward in Arduino IDE — easier path: edit `boards.txt` or use PlatformIO):

```
-DVision_Master_E213
```

Without that define, the `LCMEN2R13EFC1` driver in `heltec-eink-modules` will refuse to compile (it gates the constructor behind `#if defined(WIRELESS_PAPER) || defined(Vision_Master_E213)`).

## Required libraries (Library Manager)

- `heltec-eink-modules` by Todd Herbert
- `Adafruit GFX Library`
- `U8g2_for_Adafruit_GFX`

`LittleFS`, `WiFi`, `WebServer`, `Preferences` and the `esp_*` sleep/wifi/bt headers ship with the ESP32 core.

## Flashing

1. Connect the E213 via USB-C.
2. `Tools → Port` → pick the `cu.usbmodem*` entry.
3. `Sketch → Upload` (→ icon). First flash takes ~40 s. The board reboots automatically.

## Known gaps to confirm after the first successful boot

- [ ] Display orientation and inversion (may need `setRotation` tweak).
- [ ] Locate the VBAT ADC pin from the schematic, set `BAT_ADC_IN`, and flip `HAS_BATTERY` back to `1`.
- [ ] The E213 has a `VEXT` rail on `GPIO 18` that powers peripherals — not used by Pala One today, but worth knowing for deep-sleep current.
- [ ] The E213 has a second user button on `GPIO 21` (`BUTTON_2`) — Pala One only uses one button (`BOOT` / `GPIO 0`), so this is just available headroom.
