# Frigate LLM Vision Timeline Card

[![GitHub Release](https://img.shields.io/github/v/release/agrestisdavid/frigate-llm-vision-timeline-card?style=flat-square)](https://github.com/agrestisdavid/frigate-llm-vision-timeline-card/releases)
[![HACS Custom](https://img.shields.io/badge/HACS-Custom-orange.svg?style=flat-square)](https://hacs.xyz/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)

A custom [Lovelace](https://www.home-assistant.io/lovelace/) card for [Home Assistant](https://www.home-assistant.io/) that combines three things in a single card:

- **Frigate event timeline** — clip thumbnails, snapshots and metadata pulled from the [Frigate](https://frigate.video/) integration's media source.
- **LLM Vision enrichment** — when [LLM Vision](https://github.com/valentinfrlch/ha-llmvision) is installed, every event gets an AI-generated title and description on top of the raw label.
- **Live & VoD playback** — go2rtc-backed live streams (WebRTC / MSE / HLS / MP4 / MJPEG with automatic fallback chain) and a draggable, zoomable VoD timeline that scrubs through the full Frigate recording.

The card is one self-contained ES module — no build step, no extra dependencies beyond what HACS / Home Assistant already provide.

---

## Table of Contents

- [Feature Overview](#feature-overview)
- [Requirements](#requirements)
- [Installation](#installation)
  - [HACS (recommended)](#hacs-recommended)
  - [Manual](#manual)
- [Quick Start](#quick-start)
- [Configuration Reference](#configuration-reference)
  - [Required vs. Optional](#required-vs-optional)
  - [Cameras](#cameras)
  - [General](#general)
  - [Layout & Sizing](#layout--sizing)
  - [View Mode](#view-mode)
  - [Filters & Language](#filters--language)
  - [Label Styling](#label-styling)
  - [Livestream](#livestream)
  - [Multiview](#multiview)
  - [Advanced](#advanced)
- [Example Configurations](#example-configurations)
- [How It Works](#how-it-works)
  - [Events View](#events-view)
  - [Timeline View](#timeline-view)
  - [Live Stream](#live-stream)
  - [LLM Vision Enrichment](#llm-vision-enrichment)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)
- [Credits](#credits)

---

## Feature Overview

| Capability | Description |
| --- | --- |
| Event list | Snapshot, label chip, time, and (if available) LLM Vision title + description for each Frigate event. |
| Timeline | Scrollable, zoomable axis (24h → 1h, six zoom levels) showing event thumbnails as markers anchored to their exact timestamp. |
| Marker clusters | Overlapping events collapse into a peek-stack with a `+N` badge; hover or first tap fans them out into a scrollable popover. |
| VoD playback | Click anywhere on the timeline track to scrub the corresponding Frigate hour-VoD; click a marker to play the event clip. |
| Live stream | go2rtc-backed live view with provider fallback (WebRTC HTTP → WebRTC WS → MSE → MP4 → HLS → MJPEG). |
| HD/SD switch | Per-camera main/sub stream switching with automatic fallback if the HD stream's codec is unsupported. |
| Multiview | Grid layout with one live tile per configured camera (1–4 columns). |
| Filters | Camera, label, and date filters with sticky badges in the header. |
| LLM Vision | Optional AI-generated title and description per event, matched by camera + timestamp. |
| Visual editor | Full UI in the Lovelace card editor — no manual YAML required for any setting. |
| Localisation | German and English UI strings, auto-detected from the Home Assistant locale. |
| Auto-refresh | Optional periodic refetch of events; pauses when the tab is hidden. |
| Layout | `split` (player + list/timeline side-by-side) or `stacked` (one above the other), with a configurable desktop breakpoint. |
| Themes | All colours pulled from Home Assistant theme variables — looks at home in light and dark themes. |

---

## Requirements

| Component | Minimum | Notes |
| --- | --- | --- |
| Home Assistant | 2024.x | Tested up to current. |
| HACS | 1.34+ | Required for HACS install (otherwise install manually). |
| [Frigate integration](https://github.com/blakeblackshear/frigate-hass-integration) | 5.x | Provides the media source the card pulls events from. |
| Frigate | 0.13+ | Hour-based VoD requires Frigate 0.17+; older versions still work for events and clips. |
| go2rtc | bundled with Frigate | Required for live streams. |
| [LLM Vision](https://github.com/valentinfrlch/ha-llmvision) | optional | Without it, events still load — they just don't get titles or descriptions. |

---

## Installation

### HACS (recommended)

1. In HACS, open **Frontend** → top-right menu → **Custom repositories**.
2. Add `https://github.com/agrestisdavid/frigate-llm-vision-timeline-card` with category **Lovelace**.
3. Search for **Frigate LLM Vision Timeline Card** in the HACS frontend list and install it.
4. Reload the browser (`Ctrl + Shift + R`).
5. Add the card to a dashboard via the visual card picker — search for *frigate llm vision*.

HACS handles the Lovelace resource registration automatically.

### Manual

1. Download `frigate-llm-vision-timeline-card.js` from the [latest release](https://github.com/agrestisdavid/frigate-llm-vision-timeline-card/releases).
2. Copy it to `<config>/www/frigate-llm-vision-timeline-card/`.
3. Register it as a Lovelace resource (Settings → Dashboards → top-right menu → **Resources**):
   ```yaml
   url: /local/frigate-llm-vision-timeline-card/frigate-llm-vision-timeline-card.js
   type: module
   ```
4. Hard-reload the browser.

---

## Quick Start

Minimum working configuration — relies on auto-discovery of all Frigate cameras:

```yaml
type: custom:frigate-llm-vision-timeline-card
```

Recommended starting point with explicit cameras and live URL:

```yaml
type: custom:frigate-llm-vision-timeline-card
title: Cameras
frigate_url: http://192.168.1.50:5000
go2rtc_url: http://192.168.1.50:1984
cameras:
  driveway:
    name: Driveway
    main: driveway_hd
    sub: driveway_sd
  porch:
    name: Front Porch
    main: porch_hd
    sub: porch_sd
```

---

## Configuration Reference

All settings can be edited from the visual Lovelace card editor (**Edit Card** → tabs).

### Required vs. Optional

There are **no strictly required keys**. The card runs with an empty `{}` config and discovers cameras from the Frigate media source. Recommended for proper live streaming:

- `cameras` — at least one entry, mapping a Frigate camera ID to its main/sub go2rtc stream names.
- One of `frigate_url` / `go2rtc_url` — so the card can reach go2rtc for live playback.

Everything else is optional with sensible defaults.

### Cameras

```yaml
cameras:
  <frigate_camera_id>:
    name: <display_name>
    main: <go2rtc_main_stream>
    sub: <go2rtc_sub_stream>
```

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `<frigate_camera_id>` | object | – | Lower-case Frigate camera ID. Multiple cameras as siblings under `cameras:`. |
| `name` | string | capitalised camera ID | Display name shown in the camera filter, live picker, badge, and tile headers. |
| `main` | string | `<id>` | go2rtc stream name for **HD** playback (used when the HD/SD button is on HD). |
| `sub` | string | `main` | go2rtc stream name for **SD** playback (used by default and by multiview tiles). |

Alternative array forms also work:

```yaml
cameras:
  - id: driveway
    name: Driveway
    main: driveway_hd
    sub: driveway_sd
  - porch:
      name: Front Porch
      main: porch_hd
      sub: porch_sd
```

If `cameras` is omitted (or set to `"all"`), the card includes every camera it can find via `media-source://frigate`.

### General

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `title` | string | `"auto"` | Card title. `"auto"` uses a localised default ("Frigate Timeline" / "Frigate Zeitachse"). |
| `title_icon` | string | `null` | MDI icon shown next to the title (e.g. `mdi:cctv`). |
| `show_title` | bool | `true` | Hide the title bar entirely when set to `false`. |
| `frigate_client_id` | string | `"frigate"` | Frigate integration's client ID — only change if you renamed the HA Frigate integration instance. |
| `initial_events` | int | `10` | Events fetched on first load. |
| `events_per_load` | int | `10` | Step size for the **Load more** / **Less** buttons. |

### Layout & Sizing

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `layout` | enum | `"auto"` | `"split"` (player + list/timeline side-by-side), `"stacked"` (one above the other), or `"auto"` (split when card is wider than `desktop_breakpoint`). |
| `desktop_breakpoint` | int (px) | `800` | Width threshold used by `auto` layout. |
| `section_mode` | bool | `false` | If `true`, the card fills the surrounding HA section grid cell (height/width settings ignored). |
| `height` | string \| `"auto"` | `"auto"` | Card height fallback (e.g. `"600px"`, `"100%"`). |
| `mobile_height` | string | `null` | Card height when card width < `desktop_breakpoint`. Falls back to `height`. |
| `desktop_height` | string | `null` | Card height when card width ≥ `desktop_breakpoint`. Falls back to `height`. |
| `width` | string | `"auto"` | Card width (e.g. `"100%"`, `"800px"`, `"full"`). |

### View Mode

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `view_mode` | enum | `"events"` | `"events"` shows the classic event list. `"timeline"` shows the VoD timeline with markers. |
| `timeline_window_hours` | int (1–168) | `24` | Range covered by the timeline (in hours, looking back from now). Only used when `view_mode: "timeline"`. |

> **Note:** Timeline view requires Frigate 0.17+ for hour-based VoD playback. The timeline itself is always rendered vertically; only its placement (top/bottom vs. left/right) follows the `layout` setting.

### Filters & Language

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `show_filters` | bool | `true` | Show the camera / label / date / live-camera filter chips. |
| `language` | enum | `"auto"` | `"auto"`, `"de"`, or `"en"`. `"auto"` follows the Home Assistant locale. |
| `llm_vision` | bool | `true` | Set to `false` to skip the LLM Vision API call entirely (events show only the Frigate label). |

### Label Styling

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `label_style` | enum | `"soft"` | Chip variant: `"soft"` (tinted background), `"solid"` (filled), `"outline"` (border only). |
| `label_colors` | object | – | Custom color per Frigate label. Use `default` as a fallback. Example below. |
| `label_icons` | object | – | Custom MDI icon per Frigate label. Falls back to a built-in mapping (person → `mdi:account-alert`, car → `mdi:car-sports`, …). |

```yaml
label_colors:
  person: "#ef4444"
  car: "#3b82f6"
  dog: "#f59e0b"
  default: "#737373"
label_icons:
  person: mdi:human-greeting
  amazon: mdi:package-variant
```

### Livestream

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `live_provider` | enum | `"auto"` | `"auto"`, `"go2rtc"`, `"mjpeg"`, or `"off"`. |
| `go2rtc_modes` | string | `"webrtc,mse,mp4,hls,mjpeg"` | Comma-separated, ordered list of providers tried in sequence. Invalid tokens are silently dropped. |
| `frigate_url` | URL | `null` | Used as fallback to derive `<frigate_url>/api/go2rtc` when `go2rtc_url` is unset. |
| `go2rtc_url` | URL | `null` | Direct go2rtc base URL on the local network (e.g. `http://192.168.1.50:1984`). |
| `go2rtc_url_external` | URL | `null` | go2rtc base URL when the browser is **outside** the LAN (auto-detected via hostname). |
| `live_camera` | string | first configured | Default camera selected when the live view opens. Must match a `cameras` ID. |
| `live_autostart` | bool | `true` | Start the live stream automatically when the card mounts. |
| `show_live_button` | bool | `true` | Show the "Live" call-to-action when no clip is active. |
| `live_controls` | bool | `true` | Show the camera-name + LIVE indicator overlay on the player. |
| `live_controls_position` | enum | `"bottom"` | `"top"` or `"bottom"`. |
| `hd_sd_button` | bool | `true` | Show the HD/SD switch on the live player. |
| `hd_sd_button_position` | enum | `"top"` | `"top"` or `"bottom"`. |
| `prefer_mp4_cameras` | string[] | `[]` | List of camera IDs that should skip HLS for clip playback and use MP4 directly. The card also auto-learns this on HLS errors. |

#### Provider fallback chain

The default `go2rtc_modes` order is `webrtc,mse,mp4,hls,mjpeg`. Each provider gets a timeout window; failure cleanly falls through to the next. iOS Safari uses a slightly different default (`webrtc,hls,mp4,mjpeg`) because MSE is unsupported.

### Multiview

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `multiview` | bool | `false` | Enables the multi-camera grid (one live tile per configured camera). |
| `multiview_layout` | enum | `"auto"` | `"split"`, `"stacked"`, or `"auto"`. |
| `multiview_columns` | int (1–4) | `1` | Number of tiles per row in the grid. |

> **Mutually exclusive with timeline view.** Enabling `multiview` overrides `view_mode: "timeline"` back to `"events"` (the editor reflects this with a disabled select).

### Advanced

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `auto_refresh_seconds` | int | `0` | Periodic refetch of events. `0` disables. Pauses when the tab is hidden or while loading. |
| `dedupe_window_seconds` | int | `30` | Drop adjacent events on the same camera + label that fall within this window of each other. |

---

## Example Configurations

### Minimal — auto-discover everything

```yaml
type: custom:frigate-llm-vision-timeline-card
```

### Single-camera timeline view

```yaml
type: custom:frigate-llm-vision-timeline-card
view_mode: timeline
timeline_window_hours: 24
frigate_url: http://192.168.1.50:5000
go2rtc_url: http://192.168.1.50:1984
cameras:
  driveway:
    name: Driveway
    main: driveway_hd
    sub: driveway_sd
```

### Multi-camera with multiview

```yaml
type: custom:frigate-llm-vision-timeline-card
title: All Cameras
title_icon: mdi:cctv
multiview: true
multiview_columns: 2
multiview_layout: split
frigate_url: http://192.168.1.50:5000
go2rtc_url: http://192.168.1.50:1984
go2rtc_url_external: https://go2rtc.example.com
cameras:
  driveway: { name: Driveway, main: driveway_hd, sub: driveway_sd }
  porch: { name: Porch, main: porch_hd, sub: porch_sd }
  garden: { name: Garden, main: garden_hd, sub: garden_sd }
  garage: { name: Garage, main: garage_hd, sub: garage_sd }
```

### Custom label styling + auto-refresh

```yaml
type: custom:frigate-llm-vision-timeline-card
label_style: solid
label_colors:
  person: "#ef4444"
  car: "#3b82f6"
  dog: "#f59e0b"
  default: "#6b7280"
label_icons:
  amazon: mdi:package-variant
auto_refresh_seconds: 60
initial_events: 20
events_per_load: 20
```

### Section mode (HA Sections dashboard)

```yaml
type: custom:frigate-llm-vision-timeline-card
section_mode: true
view_mode: timeline
cameras:
  driveway: { main: driveway_hd, sub: driveway_sd }
```

### Forcing MP4 for a flaky camera

```yaml
type: custom:frigate-llm-vision-timeline-card
prefer_mp4_cameras: [driveway]
cameras:
  driveway: { main: driveway_hd, sub: driveway_sd }
```

---

## How It Works

### Events View

The default view (`view_mode: "events"`) walks the Frigate media source (`media-source://frigate`) to collect clips for each configured camera, then optionally enriches each event with the closest LLM Vision entry by timestamp + camera (90 s tolerance). Events are sorted by time descending and de-duplicated within `dedupe_window_seconds`.

Clicking a row resolves the clip via `media_source/resolve_media`, then plays it through hls.js (HLS) or directly (MP4) depending on what Frigate returns. The card automatically falls back from HLS to a direct MP4 endpoint if HLS fragments fail and remembers cameras that need MP4 across the session.

### Timeline View

Timeline view (`view_mode: "timeline"`) replaces the event list with a vertical axis covering `timeline_window_hours` of recordings.

- **Markers** are anchored to their exact timestamp on the axis. Each shows the event snapshot, label chip, time, and (if available) LLM Vision title and description.
- **Clusters** group events that fall within ~5 % of the visible range. The collapsed peek shows up to three cards plus a `+N` badge; hovering, focusing, or first-tapping the cluster expands it into a scrollable popover that fills the lane height — every overlapping event is reachable.
- **Click a marker** — opens its individual clip (same playback path as the events list).
- **Click the track** or **drag-pan** — scrubs the underlying Frigate hour VoD. The card loads `/api/frigate/<id>/vod/<YYYY-MM>/<DD>/<HH>/<camera>/index.m3u8` for the relevant hour. Sub-resources (init segment + fragments) are signed individually via `auth/sign_path` and the rewritten manifest is served as a Blob URL to hls.js. Switching hours triggers a quick reload.
- **Zoom** with the +/- buttons or the mouse wheel (anchored to the cursor). Six steps from 24h down to 1h.
- **Closing a clip** always returns to the live stream (and clears the timeline selection).

### Live Stream

The live stream is provided by go2rtc. The card tries each transport listed in `go2rtc_modes` in order, with per-mode timeouts and clean teardown on failure. If the HD stream fails to play (typically because the browser can't decode H.265), the card automatically falls back to the SD substream.

### LLM Vision Enrichment

When the LLM Vision integration is installed and `llm_vision: true` (default), the card calls `/api/llmvision/timeline/events?limit=200&days=7` once per fetch and joins the response into the Frigate events by camera and timestamp. Each enriched event gets:

- A title (e.g. *"Person detected at the front door"*).
- A free-form description used in event rows, the lightbox, and timeline marker tooltips.

When the integration is missing or returns an error, the join is silently skipped — events still display with their Frigate label.

---

## Troubleshooting

### Events show but clips won't play

- Verify `frigate_client_id` matches your HA Frigate integration's instance ID. The default `"frigate"` is correct for single-instance setups.
- Open the browser console and watch for `[FrigateLLMCard] HLS failed` warnings. The card auto-falls back to MP4; persistent failures usually mean the HA → Frigate proxy isn't reachable.
- Try adding the affected camera to `prefer_mp4_cameras` to skip HLS entirely.

### Live stream never starts

- Confirm `go2rtc_url` (LAN) or `go2rtc_url_external` (WAN) is reachable from the browser. The card detects the network from the page hostname.
- Browsers running on HTTPS pages can't connect to plain HTTP / WS go2rtc URLs — use HTTPS or expose go2rtc behind a reverse proxy.
- Check the console for `[FrigateLLMCard] webrtc failed` / `mse failed` lines — they include the rejected reason from each provider.
- Try forcing a known-good provider with `go2rtc_modes: "mse"` or `"mp4"` to isolate which transport fails.

### Timeline shows "No recording for this period"

- The selected hour has no Frigate recording. Either the camera was offline, the retention has expired, or `record:` isn't enabled in `frigate.yml`.
- For Frigate 0.17+, the path is interpreted in **local time** (matches Frigate's storage layout). Older versions may differ — the card uses Frigate 0.17 conventions.

### Auth errors when scrubbing the timeline

- The card signs every VoD sub-URL via Home Assistant's `auth/sign_path` WebSocket. If your reverse proxy strips the `?authSig=` query, signed URLs will 401.
- Watch the debug overlay underneath the player — it shows the manifest URL and the segment hls.js is currently fetching.

### LLM Vision data missing

- Confirm the integration version is recent enough to expose the `/api/llmvision/timeline/events` endpoint.
- The card matches LLM Vision events to Frigate events by camera + timestamp (±90 s). If your camera names differ between Frigate and LLM Vision, the join may miss — set `llm_vision: false` if you'd rather hide the placeholder.

### Multiview tiles are dark / missing the stream

- Each tile uses the camera's **sub** stream by default to save bandwidth. Make sure `sub` is set in the camera entry; otherwise the tile falls back to `main`.
- The HD/SD button is per-tile.

---

## Contributing

Issues and pull requests are welcome at <https://github.com/agrestisdavid/frigate-llm-vision-timeline-card>.

When reporting a bug, please include:

- Card version (visible in the browser console as `FRIGATE-LLM-VISION-TIMELINE-CARD vX.Y.Z`).
- Home Assistant version, Frigate version, and HACS version.
- The card's YAML configuration (redact any URLs you'd rather keep private).
- Browser console output captured while the issue happens.

---

## License

MIT — see [LICENSE](LICENSE).

---

## Credits

- [Frigate](https://frigate.video/) by Blake Blackshear and contributors.
- [LLM Vision](https://github.com/valentinfrlch/ha-llmvision) by Valentin Frlch.
- [hls.js](https://github.com/video-dev/hls.js) for HLS/fmp4 playback in the browser.
- [Home Assistant](https://www.home-assistant.io/) and the HACS community.
