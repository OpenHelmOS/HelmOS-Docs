# OpenHelmOS

Open-source marine control system for retrofitting onto existing vessels.

OpenHelmOS is a modular, extensible boat automation platform built around a Raspberry Pi core and ESP32-based peripheral modules. It is designed to work fully offline and be installable by anyone with basic technical skills.

The project is dual-licensed under GPL v3 and a commercial license. Community model inspired by Voron — open, contributor-driven, and hardware-agnostic.

**Website:** [openhelmos.io](https://openhelmos.io)  
**GitHub:** [github.com/openhelmos](https://github.com/openhelmos)  
**Discord:** Coming soon

---

## Product Family

| Kit | Description | Price |
|-----|-------------|-------|
| Starter Kit | Core RPi module, 4 built-in channels | ~250€ |
| Expansion Kit | 8-channel universal module | ~65€ |
| Autopilot Kit | Hydraulic pump + remote control | ~212€ |
| Environment Kit | Temperature, humidity, bilge sensors | ~67€ |
| Display Kit | 12" IP67 touchscreen display | ~354€ |

---

## Repositories

| Repo | Description |
|------|-------------|
| `helmos-core` | FastAPI backend, MQTT bridge, config system |
| `helmos-ui` | Vite + React frontend |
| `helmos-fw` | ESP32 firmware |
| `helmos-hw` | PCB schematics and enclosure files |
| `helmos-docs` | This documentation |
| `openhelmos.io` | Public website |

---

## Quick Links

- [Architecture Overview](architecture/overview.md)
- [Getting Started](software/getting-started.md)
- [Hardware Wiring](hardware/wiring.md)
- [MQTT Topics](software/mqtt-topics.md)
- [Contributing](contributing.md)
