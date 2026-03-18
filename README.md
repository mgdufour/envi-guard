# Envi Guard

**LoRaWAN IIoT dashboard for monitoring temperature and humidity in industrial electrical rooms.**

Live demo: https://mgdufour.github.io/envi-guard/

> DTO5389 Digital Transformation Technologies · Group 4 · University of Ottawa · 2026

---

## Overview

Envi Guard is a browser-based dashboard that simulates a LoRaWAN sensor network deployed across industrial electrical rooms. It demonstrates the full data pipeline — from sensor transmission through cloud processing to real-time alerting — using an interactive front-end simulation.

The dashboard visualises temperature and humidity data from three rooms (Electrical Room A, MCC Room B, and Control Room C), raises colour-coded alerts when readings cross configurable thresholds, and plots a rolling 30-point trend chart.

## System Architecture

```
Dragino LHT65  →  LoRaWAN Gateway  →  AWS IoT Core  →  AWS Lambda  →  Amazon Timestream  →  Dashboard + Alerts
```

| Layer | Technology |
|---|---|
| End device | Dragino LHT65 temperature & humidity sensor |
| Network | LoRaWAN (SF7, 2 gateways) |
| Cloud ingestion | AWS IoT Core (MQTT) |
| Processing | AWS Lambda |
| Storage | Amazon Timestream |
| Presentation | Single-page HTML dashboard |

## Features

### Sensor Simulator (Producer)
- Adjustable transmit interval (1 – 10 s)
- Per-room temperature (15 – 60 °C) and humidity (20 – 100 %RH) sliders with live mini-bar indicators
- Gaussian noise applied to each transmission to mimic real sensor variance
- Rolling raw MQTT payload log (last 50 messages) with colour-coded severity

### Application Engine (Consumer)
- Room status cards that update on each received payload
- Alert event log that records `WARNING`, `CRITICAL`, and `RESOLVED` state transitions
- Simulated email dispatch notification on critical events

### Threshold Configuration
| Metric | Warning | Critical |
|---|---|---|
| Temperature | 35 °C | 40 °C |
| Humidity | 70 %RH | 80 %RH |

Thresholds are editable at runtime and applied immediately.

### Summary Strip
- Active alert count (colour-coded: green / amber / red)
- Total MQTT messages sent
- Hottest room and its current temperature
- Session uptime

### Trend Chart
- 30-transmission rolling window per room
- Rendered with Chart.js 4.4
- Dashed critical-threshold reference line that tracks the configured value

## Tech Stack

- **HTML / CSS / JavaScript** — no build step, no dependencies beyond Chart.js
- **Chart.js 4.4.1** — CDN-loaded line chart
- **IBM Plex Sans & IBM Plex Mono** — Google Fonts
- Dark colour theme with amber / teal / red severity palette

## Simulated MQTT Payload

```json
{
  "device_id": "LHT65-A",
  "room": "Elec Room A",
  "timestamp": "2026-03-18T19:30:00.000Z",
  "temp_c": 28.14,
  "humidity_rh": 52.3,
  "battery_v": 3.2,
  "rssi": -78,
  "sf": 7
}
```

## Running Locally

No build step required — open `index.html` directly in a browser:

```bash
git clone https://github.com/mgdufour/envi-guard.git
cd envi-guard
open index.html   # macOS
# or just double-click index.html on Windows / Linux
```

## Repository Structure

```
.
└── index.html   # Self-contained single-page dashboard
```

## Course Context

This project was developed for **DTO5389 Digital Transformation Technologies** at the University of Ottawa. It serves as a proof-of-concept demonstrating how LoRaWAN sensors, cloud IoT services, and a lightweight front-end can be composed into an end-to-end environmental monitoring solution for industrial facilities.

> **Note:** The current dashboard is a front-end simulation only — no real hardware or cloud services are connected.
