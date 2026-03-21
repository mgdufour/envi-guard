# Envi Guard
### LoRaWAN-Based Environment Control and Industrial Proactive Maintenance System

**DTO5389 - Digital Transformation Technologies**  
**Group 4 · University of Ottawa · 2026**

---

## Live Demo
**Data Generator (this deliverable):**
[https://mgdufour.github.io/envi-guard/?session=rpi5-group4](https://mgdufour.github.io/envi-guard/?session=rpi5-group4)

**Live Firebase Data Feed (for consumer app):**
[https://group4-project-73093-default-rtdb.firebaseio.com/envi-guard/sessions/rpi5-group4/rooms.json](https://group4-project-73093-default-rtdb.firebaseio.com/envi-guard/sessions/rpi5-group4/rooms.json)

---

## About
Envi Guard is an IIoT simulation dashboard demonstrating a LoRaWAN-based temperature and humidity monitoring system for industrial electrical and MCC rooms. The dashboard simulates the full data pipeline from sensor to cloud to alert, writing live data to Firebase Realtime Database that a consumer application can read in real time.

This repository contains **Michel Dufour's deliverable** — the data generator and live stream component of the Group 4 project.

---

## Simulated Architecture
```
Dragino LHT65 sensors
        ↓
Cisco IXM-LPWA-900 LoRaWAN Gateways (×2, with redundancy failover)
        ↓
Firebase Realtime Database (simulating AWS IoT Core + Timestream)
        ↓
Dashboard + Consumer App + Alerts
```

---

## Features

### Data Generator
- Simulates three Dragino LHT65 temperature and humidity sensors across three industrial rooms
- Configurable transmit interval (default 5 seconds)
- Realistic sensor payload including device ID, timestamp, temperature, humidity, battery voltage, RSSI, and spreading factor
- Small random noise added to each reading to simulate real sensor variance

### Cisco Gateway Simulation
- Two Cisco IXM-LPWA-900-16-K9 LoRaWAN gateways (GW-CIS-01 and GW-CIS-02)
- Live RSSI signal strength, packet counts, ADR enabled, GPS sync active
- **Gateway offline toggle** — simulate a gateway failure and watch automatic failover to the redundant gateway
- When a gateway fails over, `failover: true` is included in the Firebase payload so consumer apps can react
- Both-gateway-offline scenario pauses the data stream and shows a connection lost warning

### Firebase Realtime Database
- Every sensor reading is written to Firebase in real time
- Two data paths per session:
  - `rooms/` — always the latest reading per room (consumer app reads this for live state)
  - `readings/` — full append log of every reading (historical data)
- Session-based paths prevent conflicts when multiple browsers have the dashboard open
- The Raspberry Pi runs a fixed session (`rpi5-group4`) as a permanent dedicated generator

### Session Management
The dashboard supports URL-based session IDs:
- **Fixed session (Pi):** `?session=rpi5-group4` — always writes to the same path
- **Random session (other browsers):** no parameter — generates a unique session ID automatically to avoid conflicts

### CSV / JSON Data Export
- Every reading is automatically logged in memory
- Configurable row limit (default 500 rows, oldest dropped when full)
- Download as CSV — ready to open in Excel, formatted with all sensor and gateway fields
- Download as JSON — structured payload format matching what AWS Timestream would store
- Progress bar shows how full the log is

### Alert System (Reference Implementation)
- Customizable warning and critical thresholds for temperature and humidity
- Alert event log showing status transitions (normal → warning → critical → resolved)
- Simulated email dispatch on critical events

---

## Firebase Payload Structure
Each reading written to Firebase includes:

```json
{
  "device_id": "LHT65-A",
  "room": "Elec Room A",
  "timestamp": "2026-03-20T14:22:01.053Z",
  "temp_c": 28.34,
  "humidity_rh": 51.9,
  "battery_v": 3.2,
  "sf": 7,
  "gateway_id": "GW-CIS-01",
  "gateway_rssi": -76,
  "failover": false,
  "gateways_online": 2,
  "status": "normal"
}
```

**Status values:** `normal`, `warning`, `critical`  
**Failover:** `true` when one gateway has failed and the other is carrying all traffic

---

## Connecting a Consumer App
Your application can read live room state from Firebase using a simple HTTP GET:

```
GET https://group4-project-73093-default-rtdb.firebaseio.com/envi-guard/sessions/rpi5-group4/rooms.json
```

For real-time push updates using the Firebase SDK:

```javascript
import { initializeApp } from "https://www.gstatic.com/firebasejs/12.11.0/firebase-app.js";
import { getDatabase, ref, onValue } from "https://www.gstatic.com/firebasejs/12.11.0/firebase-database.js";

const firebaseConfig = {
  apiKey: "AIzaSyCxyK4f2LsagBkArBYBFVtl867CiqYGPzE",
  authDomain: "group4-project-73093.firebaseapp.com",
  databaseURL: "https://group4-project-73093-default-rtdb.firebaseio.com",
  projectId: "group4-project-73093",
  storageBucket: "group4-project-73093.firebasestorage.app",
  messagingSenderId: "680924898297",
  appId: "1:680924898297:web:a2ad5a5302da4ab0145328"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

// Listen for live room updates
const roomsRef = ref(db, 'envi-guard/sessions/rpi5-group4/rooms');
onValue(roomsRef, (snapshot) => {
  const data = snapshot.val();
  console.log('Live room data:', data);
  // Check for failover
  Object.values(data).forEach(reading => {
    if (reading.failover) {
      console.warn('Gateway failover detected — only 1 gateway online');
    }
  });
});
```

---

## Hardware (Simulated)
| Component | Model | Details |
|---|---|---|
| Sensors | Dragino LHT65 | Temperature & humidity, LoRaWAN Class A, 2400mAh battery |
| Gateways | Cisco IXM-LPWA-900-16-K9 | LoRaWAN Class A/B/C, ADR, GPS sync, TDOA geolocation |
| Cloud | Firebase Realtime Database | Simulating AWS IoT Core + Lambda + Timestream |
| Hosting | GitHub Pages | mgdufour.github.io/envi-guard |
| Generator hardware | Raspberry Pi 5 | Runs the dashboard 24/7 as a dedicated data generator |

---

## Deployment
The dashboard is a single self-contained `index.html` file with no build step or dependencies required.

**To run locally:**
Just open `index.html` in any browser. Add `?session=your-session-id` to the URL to write to a named Firebase path.

**To deploy updates:**
```bash
git add .
git commit -m "Your message"
git push
```
GitHub Pages automatically serves the updated file within a few minutes.

---

## Team
| Name | Role | Responsibilities |
|---|---|---|
| Dandan Cao | Research & Technical Analyst | Research environmental risks, sensor selection, problem statement |
| Carlos Del Rio | Lead Solution Architect | Proposed solution, methodology, system architecture |
| Jessy Sindayihebura | Project Manager | Timeline, team roles, proposal compilation |
| Michel Dufour | Risk & Validation Specialist / Data Generator | Risk register, data generator, Firebase live stream, GitHub Pages deployment |
| Laxmi Badri | Cloud & Data Integration Engineer | Backend development, cloud services, API and dashboard integration |

---

## Technologies Used
- HTML5, CSS3, JavaScript (vanilla — no framework)
- Firebase Realtime Database (live data stream)
- Chart.js (temperature trend visualization)
- GitHub Pages (hosting)
- Raspberry Pi 5 (dedicated 24/7 data generator)
- LoRaWAN protocol (simulated)
- Cisco IXM-LPWA-900 gateways (simulated)
- Dragino LHT65 sensors (simulated)

---

*Simulation only — no real hardware connected. All sensor readings, gateway metrics, and network data are generated programmatically to demonstrate system behaviour.*
