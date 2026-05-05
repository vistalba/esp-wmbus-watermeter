# ESPHome wMBus Water Meter

ESPHome configuration for reading encrypted **Wireless M-Bus (wMBus)** water meters using the **M5Stack Unit C6L** (ESP32-C6 + SX1262 LoRa transceiver). Values are transmitted directly to **Home Assistant** via the native ESPHome API — no MQTT broker required.

## Hardware

| Component | Description | Link |
|---|---|---|
| M5Stack Unit C6L | ESP32-C6 + SX1262, 868 MHz LoRa, OLED, RGB LED | [Docs](https://docs.m5stack.com/en/unit/Unit_C6L) |
| Water Meter | Sensus iPERL (wMBus T1, encrypted) | — |

### Why Unit C6L?
- ESP32-C6 with native Wi-Fi 6
- Integrated SX1262 LoRa transceiver (868–923 MHz)
- No external radio module required
- Compact form factor

## Features

- Reads encrypted wMBus T1 telegrams
- Decrypts AES-128 CBC encrypted payloads
- Exposes sensors to Home Assistant:
  - Water consumption (m³)
  - Flow rate (m³/h)
  - Radio signal strength (RSSI)
- Diagnostic sensors: WiFi signal, uptime, free heap, IP address, temperature
- OTA updates via ESPHome
- Safe mode / restart buttons

## Requirements

- [ESPHome](https://esphome.io) 2026.4.0 or newer
- Home Assistant with ESPHome integration
- wMBus meter with known AES key and meter ID
- [SzczepanLeon/esphome-components](https://github.com/SzczepanLeon/esphome-components) (loaded automatically via `external_components`)

## Pin Map (M5Stack Unit C6L / Stamp C6LoRa)

| Signal | GPIO |
|---|---|
| SPI CLK | GPIO20 |
| SPI MOSI | GPIO21 |
| SPI MISO | GPIO22 |
| SX1262 CS | GPIO23 |
| SX1262 BUSY | GPIO19 |
| SX1262 IRQ | GPIO7 |
| I2C SDA (Expander) | GPIO10 |
| I2C SCL (Expander) | GPIO8 |

> **Note:** The SX1262 reset, antenna switch (ANT_SW) and LNA enable (LNA_EN) are controlled via the onboard PI4IOE5V6408 I2C GPIO expander — not directly wired to ESP32 GPIOs. ESPHome initialises these automatically via the I2C bus on boot.

## Configuration

### secrets.yaml

Add the following entries to your `secrets.yaml`:

```yaml
wifi_ssid: "YourWiFiNetwork"
wifi_password: "YourWiFiPassword"
fallback_password: "YourFallbackPassword"
ota_password: "YourOTAPassword"
esp-watermeter_api_key: "your-base64-encoded-api-key=="

# Water meter specific
watermeter_id: "0x12345678"       # Your meter ID in hex, e.g. 0x21101234
watermeter_key: "00112233445566778899AABBCCDDEEFF"  # AES-128 key, 32 hex chars
```

#### How to find your meter ID and key
- **Meter ID:** Printed on the physical meter label (8 digits). Convert to hex: `21101234` → `0x21101234`
- **AES Key:** Provided by your water utility or installer. 32 hexadecimal characters.
- You can verify decryption at [wmbusmeters.org/analyze](https://wmbusmeters.org/analyze)

### Supported meter types

This config uses `type: iperl` (Sensus iPERL). For other meters, check the supported drivers:
- [wmbusmeters driver list](https://github.com/wmbusmeters/wmbusmeters/#running-without-config-files-good-for-experimentation-and-test)

Change the `type` in `secrets.yaml` or directly in the YAML accordingly.

## Installation

### First flash (via USB)

The Unit C6L cannot be auto-detected by the ESPHome dashboard on first flash.

1. In ESPHome Dashboard → your device → **"Manual Download"** → download the `.bin` file
2. Put the device into download mode: **hold the reset button for 3 seconds** until the green LED turns red
3. Open [web.esphome.io](https://web.esphome.io) in Chrome/Edge
4. Click **Connect** → select COM port → **Install** → select your `.bin` file

### Subsequent updates

OTA updates work normally via the ESPHome dashboard after the first flash.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| No frames received | Antenna not connected or wrong antenna port | Connect 868 MHz antenna (not 2.4 GHz) |
| Frames received, meter not matched | Wrong `meter_id` format | Use hex format: `0x21101234` |
| Decryption failed | Wrong AES key | Verify key at wmbusmeters.org/analyze |
| Build fails with CMake error | Corrupt build cache | ESPHome Dashboard → Clean Build Files |
| `esp32` component conflict | External component overrides internal | Ensure `components:` list is explicit in `external_components` |

### Enable verbose logging temporarily

```yaml
logger:
  level: VERBOSE
```

Check logs for lines like:
```
(meter) Volume total_m3 decoded m3 value 258.329
'Water Consumption': Received new state 258.329
```

## References

- [ESPHome Documentation](https://esphome.io)
- [SzczepanLeon/esphome-components](https://github.com/SzczepanLeon/esphome-components)
- [M5Stack Unit C6L Documentation](https://docs.m5stack.com/en/unit/Unit_C6L)
- [wmbusmeters project](https://github.com/wmbusmeters/wmbusmeters)
- [wmbusmeters online analyzer](https://wmbusmeters.org/analyze)

## License

MIT
