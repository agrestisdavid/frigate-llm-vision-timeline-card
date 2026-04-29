# Frigate LLM Vision Timeline Card

A custom Lovelace card for Home Assistant that shows a timeline of [Frigate](https://frigate.video/) events enriched with [LLM Vision](https://github.com/valentinfrlch/ha-llmvision) descriptions.

## Features

- Timeline view of Frigate camera events
- LLM Vision AI descriptions per event
- Live stream view (WebRTC / MSE / HLS / MJPEG via go2rtc)
- VoD timeline with recording playback
- Filter by camera, label, and date
- Multi-camera multiview support
- German & English UI

## Installation via HACS

1. In HACS, go to **Frontend** → **+ Explore & Download Repositories**
2. Search for **Frigate LLM Vision Timeline Card**
3. Download and add the resource in your Lovelace configuration

## Manual Installation

Copy `frigate-llm-vision-timeline-card.js` to your `www/` folder and add it as a Lovelace resource:

```yaml
resources:
  - url: /local/frigate-llm-vision-timeline-card.js
    type: module
```

## Configuration

```yaml
type: custom:frigate-llm-vision-timeline-card
frigate_url: http://your-frigate-host:5000
go2rtc_url: http://your-frigate-host:1984
cameras:
  your_camera_name:
    name: My Camera
```

See the full configuration reference in the [wiki](../../wiki).

## Version

`0.30.0`
