# E-Ink Now Playing

An ESPHome-powered "now playing" display for YouTube Music using the ELECROW CrowPanel ESP32-S3 4.2" e-ink screen. Shows album art, track title, artist name, and playback state pulled from Home Assistant, with physical controls for playback.

![CrowPanel 4.2" E-Paper Display](https://www.elecrow.com/media/catalog/product/cache/b6b9577937e6a96f50e53ddc42983628/h/a/hardware_overview_of_4.2_inch_esp32_hmi_e-paper_display.png)

## Features

- **Album art** downloaded and rendered in real-time (binary black/white, inverted)
- **Track info** — title, artist, and playback state (playing/paused/idle)
- **Physical controls** via rotary dial and buttons
- **Event-driven display** — only refreshes when track changes, no polling
- **Auto-reconnect** to Home Assistant via native API

## Hardware

| Component | Detail |
|-----------|--------|
| **Display** | ELECROW CrowPanel 4.2" e-ink, 400x300, SSD1683 |
| **MCU** | ESP32-S3-WROOM-1-N8R8 (8MB flash, 8MB PSRAM) |
| **Framework** | ESP-IDF (required for JPEG decoding) |
| **Display driver** | [semvis123/esphome-crowpanel-4.2-epaper](https://github.com/semvis123/esphome-crowpanel-4.2-epaper) |

### Pin Mapping

| Function | GPIO |
|----------|------|
| SPI CLK | 12 |
| SPI MOSI | 11 |
| Display CS | 45 |
| Display DC | 46 |
| Display RST | 47 |
| Display BUSY | 48 |
| Display PWR | 7 |

### Controls

The CrowPanel has a rotary dial (left side) and two buttons:

| Control | GPIO | Action |
|---------|------|--------|
| MENU button | 2 | Refresh display |
| Rotary Up | 6 | Previous track |
| Rotary Press | 5 | Play / Pause |
| Rotary Down | 4 | Next track |
| EXIT button | 1 | Stop playback |

## Display Layout

```
+--------+----------------------------------+
|        |    +------------------+          |
|Refresh |    |                  |          |
|        |    |   ALBUM ART      |          |
| Prev   |    |   200 x 200      |          |
|Play/Pause   |   (inverted B&W) |          |
| Next   |    +------------------+          |
|        |      Track Title (bold)          |
| Stop   |      Artist Name                |
|        |        Playing                   |
+--------+----------------------------------+
```

## Prerequisites

- [Home Assistant](https://www.home-assistant.io/) with [ESPHome add-on](https://esphome.io/)
- [ytube_music_player](https://github.com/KoljaWindeler/ytube_music_player) custom component installed
- A media player entity (e.g., `media_player.home_group`) that receives YouTube Music playback

## Setup

### 1. Configure secrets

Add to your ESPHome `secrets.yaml`:

```yaml
wifi_ssid: "your-wifi-ssid"
wifi_password: "your-wifi-password"
```

### 2. Update entity and URL

In `youtube-music-jukebox.yaml`, update these to match your setup:

- **Entity ID** — replace all instances of `media_player.home_group` with your media player entity
- **HA base URL** — replace `http://YOUR_HA_URL:8123` with your Home Assistant URL (only needed if `entity_picture` returns a relative path)

### 3. Flash

**First time (USB required):**
1. Copy the YAML to your ESPHome config directory
2. In the ESPHome dashboard, click Install > Plug into the computer running ESPHome Dashboard
3. Select the COM port for your CrowPanel

**Subsequent updates (OTA):**
```
esphome run youtube-music-jukebox.yaml
```

### 4. Enable permissions in Home Assistant

After the device appears in HA:
1. Go to **Settings > Devices & Services > ESPHome**
2. Click **Configure** on the device
3. Enable **"Allow the device to perform Home Assistant actions"**

## How It Works

1. Four `homeassistant` text sensors subscribe to `media_title`, `media_artist`, `entity_picture`, and player state from your media player entity
2. When `entity_picture` changes, the URL is passed to the `online_image` component which downloads the JPEG album art
3. The image is resized to 200x200, converted to binary (black/white), and rendered inverted on the e-ink display
4. Track title and artist are displayed below the art with automatic truncation for long text
5. Physical button presses call Home Assistant services (`media_player.media_previous_track`, `media_player.media_next_track`, `media_player.media_play_pause`, `media_player.media_stop`)

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Display shows "Unknown Title" | Check that your media player entity has `media_title` attribute in Developer Tools > States |
| Album art shows black box | Verify `entity_picture` URL is accessible; check ESPHome logs for download errors |
| No data after boot | Restart Home Assistant — state subscriptions sometimes need an HA restart to activate |
| Device boot loops | Check that GPIO7 (display power) is configured; ensure PSRAM is enabled |
| Album art URL error | If `entity_picture` is a full URL (starts with `http`), it's used directly; relative paths get the HA base URL prepended |

## Gotchas

- **GPIO7 must be HIGH before SPI** — handled in `on_boot` at priority -100
- **PSRAM required** — without octal PSRAM, image downloads will OOM
- **ESP-IDF framework required** — Arduino framework is missing the `esp-dsp` library needed for JPEG decoding
- **First flash must be USB** — OTA only works after ESPHome firmware is installed

## License

MIT
