# ESPHome reTerminal E1002

ESPHome configuration for the **reTerminal E1002**, an ESP32-S3 based device with a large e-ink display. It turns the reTerminal into a Home Assistant dashboard that periodically fetches a screenshot from a Lovelace view and renders it on the e-paper display.

## Hardware

| Component | Details |
|---|---|
| **MCU** | ESP32-S3 (with PSRAM, octal mode, 80 MHz) |
| **Display** | 7.3" Spectra E6 color e-ink (SPI), 6 colors |
| **Sensors** | SHT4x (temperature & humidity), ADC battery voltage |
| **Buzzer** | LEDC PWM on GPIO45 |
| **Buttons** | 3 physical keys (GPIO3, GPIO4, GPIO5) |

## Features

- Fetches a dashboard image from Home Assistant via HTTP and displays it on the e-ink screen
- Automatic refresh every 30 minutes (paused between 01:00–06:00 to avoid unnecessary updates)
- Battery monitoring with voltage-to-percentage calibration
- Buzzer service (`play_song`) callable from Home Assistant automations
- OTA update support
- Manual display refresh via button press or Home Assistant switch
- Bluetooth (BLE proxy) available but disabled – can be enabled if needed

## Files

| File | Description |
|---|---|
| `reTerminal_e1002.yaml` | Production ESPHome configuration – all credentials externalized via secrets |

---

## Setup

### 1. Prerequisites

- [ESPHome](https://esphome.io/guides/installing_esphome) installed (CLI or Dashboard)
- USB-to-serial adapter (e.g. CP2102) or direct USB cable to the ESP32-S3
- The **Puppet** Home Assistant add-on is required to generate dashboard screenshots for the display. It runs a screenshot server on port 10000 that the reTerminal fetches images from. Installation and configuration instructions can be found on the [Puppet GitHub page](https://github.com/balloob/home-assistant-addons/tree/main/puppet).

### 2. Erase Flash

> **Important:** When flashing for the first time, or after flash issues, the ESP32-S3 memory must be fully erased beforehand. Skipping this step can cause flashing to fail.

First, identify the correct serial port:

```bash
# macOS / Linux
ls /dev/tty.*

# Windows
# Device Manager → Ports (COM & LPT)
```

Erase the flash using `esptool`:

```bash
pip install esptool   # if not already installed

esptool --chip esp32s3 --port /dev/tty.YOUR-PORT erase-flash
```

Expected output on success:

```
Connected to ESP32-S3 on /dev/tty.usbserial-110:
Chip type:    ESP32-S3 (revision v0.2)
Features:     Wi-Fi, BT 5 (LE), Dual Core, 240MHz, Embedded PSRAM 8MB
Flash memory erased successfully in 46.2 seconds.
Hard resetting via RTS pin...
```

Power-cycle the device before proceeding to the next step.

### 3. Create secrets.yaml

Create a file named `secrets.yaml` in the same folder as `reTerminal_e1002.yaml`:

```yaml
# Wi-Fi
wifi_ssid: "Your-WiFi-Name"
wifi_password: "Your-WiFi-Password"

# ESPHome API key
reterminal_api_key: "your-base64-api-key"

# OTA password
reterminal_ota_password: "your-ota-password"

# Home Assistant dashboard screenshot URL
# Format: http://<HA-IP>:<port>/lovelace/<view-name>?viewport=800x480&format=png
reterminal_dashboard_url: "http://192.168.x.x:10000/lovelace/reterminal?viewport=800x480&format=png"
```

**Where to get the values:**

| Secret | Source |
|---|---|
| `wifi_ssid` / `wifi_password` | Your Wi-Fi credentials |
| `reterminal_api_key` | Generate in ESPHome: menu → "Secrets", or manually via `openssl rand -base64 32` |
| `reterminal_ota_password` | Freely chosen – use a strong password |
| `reterminal_dashboard_url` | IP address of your Home Assistant + port of the screenshot service (e.g. `browsermod` or `ha-lovelace-card-screenshot`) |

> `secrets.yaml` is listed in `.gitignore` and will never be uploaded to the repository automatically.

### 4. Flash

```bash
esphome run reTerminal_e1002.yaml
```

First flash via USB, subsequent updates via OTA over the network.

---

## Display Refresh

The display refreshes automatically every 30 minutes. The quiet period from 01:00–06:00 suppresses unnecessary updates overnight. The display can also be refreshed manually at any time:

- Press **Key 3 (GPIO5)**
- Toggle the **"Refresh" switch** in Home Assistant

---

## Bluetooth (optional)

BLE tracker and Bluetooth proxy are available in the config but commented out. To enable them, uncomment the following blocks in `reTerminal_e1002.yaml`:

```yaml
esp32_ble_tracker:
  scan_parameters:
    active: true

bluetooth_proxy:
  active: true
```

> **Note:** Bluetooth significantly increases power consumption as Wi-Fi and BLE must operate simultaneously.
