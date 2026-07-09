# Findus Firmware

Firmware releases for Findus, a Swedish language tutor built on the ESP32-S3 platform.

Findus is a conversational friend for kids who need more Swedish in their lives. Press the button, speak Swedish, and Findus (the cat from Pettson och Findus) responds with questions, encouragement, and gentle corrections. Messages prefixed with "Morfar" are routed to Slack so grandpa can stay in the loop.

## Hardware

- Board: Spotpear ESP32-S3 1.54" MUMA (ESP32-S3-WROOM-1-N16R8)
- Audio: ES8388 codec, MEMS microphone, 1W speaker
- Display: 1.54" IPS LCD 240x240 (ST7789)
- Based on the [xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) framework (v2.2.6 fork)

## Features

- Swedish speech recognition and synthesis with three distinct voices (Findus, Morfar, System)
- Conversational Swedish cat persona that keeps kids talking
- Slack integration for family messaging ("Morfar: ..." routes to grandpa)
- Remote config via Slack canvas (no reflash needed to change behavior)
- OTA firmware updates from this repo
- Crash reporting to Slack
- Voice activity detection with natural pause handling

## OTA Updates

Devices check this repo's latest release on boot. If a newer version is found, the firmware is downloaded and flashed automatically with rollback protection.

## Flashing Manually

```bash
# Full flash (first time or recovery)
esptool.py --chip esp32s3 --port /dev/cu.usbmodem21201 --baud 460800 \
  write_flash 0x0 merged-binary.bin

# App-only flash (preserves WiFi config)
esptool.py --chip esp32s3 --port /dev/cu.usbmodem21201 --baud 460800 \
  write_flash 0x20000 xiaozhi.bin
```

## Configuration

Findus loads configuration from three sources (highest priority first):

1. **SD card** (`/sdcard/findus_config.json`) -- per-device settings (child name, WiFi, Slack channel)
2. **Slack canvas** -- shared system prompt, conversation seeds, cultural references (editable from phone)
3. **Built-in defaults** -- compiled into firmware as fallback

## Creating a Release

```bash
gh release create v1.1.0 build/merged-binary.bin \
  --repo catellie/findus-firmware \
  --title "v1.1.0" \
  --notes "Description of changes"
```
