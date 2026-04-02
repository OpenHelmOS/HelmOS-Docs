# Architecture Overview

OpenHelmOS is built around three layers: a Raspberry Pi core, ESP32 peripheral modules, and a React-based UI. All communication between layers uses MQTT.

---

## System Diagram

```
┌─────────────────────────────────────────────┐
│              helmos-ui (React)              │
│         Vite dev server / kiosk mode        │
└─────────────────┬───────────────────────────┘
                  │ HTTP / WebSocket
┌─────────────────▼───────────────────────────┐
│            helmos-core (FastAPI)            │
│         Config system / API endpoints       │
└─────────────────┬───────────────────────────┘
                  │ MQTT
┌─────────────────▼───────────────────────────┐
│           Mosquitto MQTT Broker             │
└──────┬──────────────────────────┬───────────┘
       │ HelmOS CAN (500kbit/s)   │ NMEA 2000 (250kbit/s)
┌──────▼──────┐            ┌──────▼──────┐
│ ESP32       │            │ Lowrance    │
│ Modules     │            │ Chartplotter│
│ (HelmOS-fw) │            │ Other NMEA  │
└─────────────┘            └─────────────┘
```

---

## Key Design Principles

**MQTT as the universal bus**
All data flows through Mosquitto. The UI, core, and firmware never communicate directly — everything is publish/subscribe. This means any component can be replaced or mocked independently.

**CAN bus is the lowest layer**
ESP32 modules communicate with the RPi over CAN bus. Above MQTT, nothing knows about CAN. The UI and FastAPI only see MQTT topics regardless of whether data comes from a real sensor or a mock script.

**Dual CAN bus separation**
HelmOS-native CAN (500kbit/s) and NMEA 2000 (250kbit/s) are kept strictly separate. NMEA 2000 devices do not integrate into HelmOS internals — they are bridged via Signal K or canboat.

**Offline-first**
The system is fully functional without internet. The RPi operates as a WiFi access point in normal use. Internet connectivity is only needed for updates and is established via a phone hotspot.

**Modular by design**
Modules announce themselves at boot via MQTT. The core detects new modules, requests their config schema, and prompts the user to complete setup. The UI activates the relevant pages automatically.

---

## Services and Boot Order

All services are managed by systemd and start automatically at boot.

```
mosquitto → helmos-core → helmos-ui → lightdm → chromium (kiosk)
```

| Service | Description | Port |
|---------|-------------|------|
| `mosquitto` | MQTT broker | 1883 |
| `helmos-core` | FastAPI backend | 8000 |
| `helmos-ui` | Vite React frontend | 5173 |
| `lightdm` | Display manager (autologin) | — |
| `vncserver-x11-serviced` | Remote desktop | 5900 |

---

## Related Documents

- [Software Architecture](software.md)
- [Network Architecture](network.md)
- [Module System](modules.md)
