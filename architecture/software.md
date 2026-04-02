# Software Architecture

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vite + React + TypeScript + Tailwind + shadcn/ui |
| Backend | FastAPI + uvicorn |
| Messaging | Mosquitto MQTT + paho-mqtt |
| Data | InfluxDB (time-series), config.json (settings) |
| Automation | Node-RED |
| Marine data | Signal K, canboat |
| Runtime | Raspberry Pi 5 (8GB), Ubuntu / Raspberry Pi OS |

---

## helmos-core

FastAPI application that acts as the bridge between MQTT and the UI.

**Responsibilities:**
- Expose REST endpoints for UI interactions
- Bridge MQTT messages to WebSocket clients
- Read and write `config.json`
- Manage module discovery and config schema
- Trigger OTA updates for UI and firmware

**Directory structure:**
```
helmos-core/
├── main.py              # FastAPI app entry point
├── config.json          # Runtime configuration
├── modules/             # Per-module routers and logic
│   ├── autopilot.py
│   ├── lights.py
│   └── environment.py
├── requirements.txt
└── venv/
```

**Running as a service:**
```
/etc/systemd/system/helmos-core.service
ExecStart=/home/juzo/helmos-core/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
```

---

## helmos-ui

React SPA served by Vite dev server in development and kiosk mode on the display.

**Design language:** Dark MBUX-inspired theme. Near-black background, blue/turquoise accents. Fonts: Exo 2 (UI) + Share Tech Mono (data values).

**Module-based routing:**
```
helmos-ui/src/
├── modules/
│   ├── autopilot/
│   │   ├── AutopilotPage.tsx
│   │   ├── AutopilotDashboard.tsx
│   │   ├── AutopilotSettings.tsx
│   │   ├── components/
│   │   │   ├── HeadingDial.tsx
│   │   │   ├── RudderIndicator.tsx
│   │   │   └── PIDTuner.tsx
│   │   └── hooks/
│   │       └── useAutopilot.ts
│   ├── lights/
│   └── environment/
├── components/ui/       # shadcn/ui base components
└── App.tsx
```

Navigation is generated dynamically from the active module list in `config.json`. Pages for inactive modules are hidden automatically.

**Running as a service:**
```
/etc/systemd/system/helmos-ui.service
ExecStart=/home/juzo/.nvm/versions/node/v20.20.2/bin/node
          /home/juzo/helmos-ui/node_modules/.bin/vite
          --host 0.0.0.0 --port 5173
```

> **Note:** systemd does not inherit nvm environment. Both Node and vite binary paths must be hardcoded with full absolute paths.

---

## helmos-fw

ESP32 firmware for peripheral modules.

**At boot, each module:**
1. Connects to WiFi
2. Publishes an announce message to `helmos/modules/announce`
3. Subscribes to its own config topic
4. Sends a heartbeat every 30 seconds

**OTA updates** are delivered wirelessly from helmos-core using ESP-IDF OTA or Arduino OTA. No physical access required.

---

## Config System

All configuration lives in a single `config.json` file on the RPi. It is the source of truth for the entire system.

See [Config documentation](../software/config.md) for full schema.

---

## Kiosk Mode

On the display module, Chromium runs in kiosk mode and opens helmos-ui automatically at boot.

```
~/.config/autostart/helmos-kiosk.desktop
Exec=bash -c "sleep 5 && chromium-browser --kiosk --noerrdialogs
     --disable-infobars --no-first-run
     --disable-restore-session-state http://localhost:5173"
```

Mouse cursor is hidden with `unclutter`. LightDM autologin is configured for the `juzo` user with graphical.target as the default systemd boot target.
