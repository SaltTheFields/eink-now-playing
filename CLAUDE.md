# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ESPHome-based "now playing" e-ink display for YouTube Music. Pulls track metadata and album art from the `ytube_music_player` Home Assistant integration and renders it on an ELECROW CrowPanel 4.2" e-ink screen with physical button controls.

## Hardware

- **Board**: ELECROW CrowPanel ESP32-S3 4.2" e-ink (400x300, SSD1683 dual-driver)
- **MCU**: ESP32-S3-WROOM-1-N8R8 (8MB flash, 8MB PSRAM)
- **SPI pins**: CS=45, DC=46, RST=47, BUSY=48, MOSI=11, SCK=12, PWR=7
- **Buttons**: PREV=GPIO6, NEXT=GPIO4, OK=GPIO5, HOME=GPIO2, EXIT=GPIO1 (all active LOW, INPUT_PULLUP)

## Build & Flash

This is an ESPHome YAML project — there is no local build toolchain. The YAML is compiled and flashed via the ESPHome dashboard (typically running as a Home Assistant add-on).

- **Validate YAML locally** (if ESPHome CLI is installed): `esphome config youtube-music-jukebox.yaml`
- **Compile without flashing**: `esphome compile youtube-music-jukebox.yaml`
- **Flash OTA**: `esphome run youtube-music-jukebox.yaml`
- **View logs**: `esphome logs youtube-music-jukebox.yaml`

Secrets (`wifi_ssid`, `wifi_password`, `api_encryption_key`, `ota_password`) must be defined in a `secrets.yaml` file in the same directory or in the ESPHome config directory.

## Architecture

Single file: `youtube-music-jukebox.yaml`. Key sections:

- **External component**: `github://semvis123/esphome-crowpanel-4.2-epaper` — third-party SSD1683 driver required for this specific panel's dual-driver setup.
- **HA text sensors**: Four sensors pull `media_title`, `media_artist`, `entity_picture`, and player state from `media_player.ytube_music_player`.
- **Online image**: Downloads album art as JPEG, applies Floyd-Steinberg dithering to grayscale, resizes to 230x230.
- **Display lambda**: C++ lambda renders art (centered top) + text (bottom third). Event-driven only (`update_interval: never`).
- **URL construction**: `entity_picture` from HA is a relative path — the `on_value` lambda prepends the HA base URL (`http://YOUR_HA_URL:8123`).

## Key Gotchas

- **GPIO7 must be HIGH before any SPI activity** — handled in `on_boot` at priority -100.
- **PSRAM must be enabled** (octal mode) or image downloads will OOM.
- **entity_picture is a relative URL** — always needs the HA base URL prepended.
- **Album art auth**: The HA proxy URL includes a token; if 401 errors appear, a Long-Lived Access Token may need to be added to HTTP headers.
