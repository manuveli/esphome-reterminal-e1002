# ESPHome reTerminal E1002

ESPHome configuration for the **reTerminal E1002**, an ESP32-S3 based device with a large e-ink display. It turns the reTerminal into a Home Assistant dashboard that periodically fetches and renders a screenshot from a Lovelace view onto the e-paper screen.

## Hardware

- **MCU:** ESP32-S3 (with PSRAM in octal mode)
- **Display:** 7.3" Spectra E6 color e-ink (SPI)
- **Sensors:** SHT4x (temperature & humidity), ADC battery voltage
- **Buzzer:** LEDC PWM on GPIO45
- **Buttons:** 3 physical keys (GPIO3, GPIO4, GPIO5)

## Features

- Fetches a dashboard image from Home Assistant via HTTP and displays it on the e-ink screen
- Hourly automatic refresh (skipped between 1:00–6:00 AM to avoid unnecessary updates)
- Battery level monitoring with voltage-to-percentage calibration
- Bluetooth Low Energy (BLE) proxy for Home Assistant
- Buzzer service (`play_song`) callable from Home Assistant automations
- OTA update support
- Manual display refresh via button press or Home Assistant switch

## Files

| File | Description |
|------|-------------|
| `reterminal-e1002-base` | Main ESPHome configuration (7.3" Spectra E6 color e-ink, PNG/RGB565, BLE proxy, buzzer service) |
| `reterminal-e1002-template` | Alternative/template configuration (Waveshare 7.50inv2 b/w e-ink, BMP/binary, no BLE proxy) |

## Getting Started

1. Copy one of the configuration files and adjust WiFi credentials, API keys, and the dashboard URL.
2. Flash the ESP32-S3 using ESPHome (`esphome run <config-file>`).
3. The device will connect to your Home Assistant instance and begin displaying the configured Lovelace view.
