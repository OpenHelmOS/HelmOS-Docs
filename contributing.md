# Contributing to OpenHelmOS

OpenHelmOS is an open-source project. Contributions of all kinds are welcome — code, documentation, hardware designs, and testing.

---

## Community

**Discord:** Coming soon  
**GitHub:** [github.com/openhelmos](https://github.com/openhelmos)

---

## Repositories

| Repo | What to contribute |
|------|--------------------|
| `helmos-core` | FastAPI endpoints, MQTT logic, config system |
| `helmos-ui` | React components, UI pages, design improvements |
| `helmos-fw` | ESP32 firmware for existing and new module types |
| `helmos-hw` | PCB designs, enclosure files, wiring diagrams |
| `helmos-docs` | Documentation improvements and corrections |

---

## Development Setup

1. Fork the relevant repository
2. Clone your fork locally
3. Follow the getting started guide for your platform
4. Create a feature branch: `git checkout -b feature/my-feature`
5. Make your changes
6. Submit a pull request with a clear description

---

## Adding a New Module Type

New module types require changes in three repositories:

**helmos-fw** — Create firmware for the new ESP32 module. Implement the announce message with the correct `config_schema` and `ui` fields. Follow the existing module structure.

**helmos-core** — Add a new router in `modules/` if the module needs custom API endpoints beyond the generic config push.

**helmos-ui** — Add a new folder under `src/modules/` with the module page, settings section, and dashboard widget. Register the route in the nav config.

---

## Coding Standards

**Python (helmos-core)**
- Follow PEP 8
- Use type hints
- FastAPI routers for each module

**TypeScript (helmos-ui)**
- Strict TypeScript
- shadcn/ui components for consistency
- MBUX dark theme — near-black background, blue/turquoise accents
- Fonts: Exo 2 for UI, Share Tech Mono for data values

**C++ (helmos-fw)**
- Arduino framework or ESP-IDF
- Implement heartbeat, announce, and config subscribe for every module
- Handle CAN bus errors gracefully

---

## Licensing

OpenHelmOS is dual-licensed:

- **GPL v3** — for open-source use
- **Commercial license** — for integration into proprietary products

By contributing, you agree that your contributions may be licensed under both licenses.

---

## Hardware Designs

PCB designs and enclosure files are in `helmos-hw`. Designs should be export-ready for manufacturing (Gerber files for PCBs, STL + STEP for enclosures).

Enclosures are designed for 3D printing. Prototype in PLA, production in ASA. Include EPDM O-ring groove and pressure equalization valve mount where applicable.
