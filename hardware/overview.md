# Hardware Overview

## Core Hardware

| Component | Model | Purpose |
|-----------|-------|---------|
| SBC | Raspberry Pi 5 (8GB) | Main computer |
| Storage | NVMe SSD via HAT+ | OS and data |
| CAN interface | MCP2515 breakout (x2) | HelmOS CAN + NMEA 2000 bridge |
| Serial | MAX3232 breakout | NMEA 0183 |
| Cooling | RPi 5 Active Cooler | Thermal management |

### HAT Stacking Note

The RPi 5 Active Cooler, NVMe HAT+, and MCP2515 HATs conflict physically and electrically. **Use MCP2515 breakout modules** wired directly to GPIO with distinct CS and INT pins:

| Interface | CS Pin | INT Pin |
|-----------|--------|---------|
| HelmOS CAN | CE0 (GPIO8) | GPIO25 |
| NMEA 2000 bridge | CE1 (GPIO7) | GPIO24 |

---

## Module Hardware

Each HelmOS module is built around an ESP32 with CAN transceiver.

| Component | Model | Purpose |
|-----------|-------|---------|
| MCU | ESP32 | Main controller |
| CAN transceiver | SN65HVD230 | CAN bus interface |
| Heading sensor | AS5048A | Magnetic absolute encoder (rudder angle) |
| Stepper driver | TMC2209 | Motor control |
| Display | GC9A01 | Round status display |

---

## Enclosures

Enclosures are 3D printed. Prototypes in PLA, production versions in ASA for UV and heat resistance. Sealed with EPDM O-ring and pressure equalization valve.

Display module: 12" 1000nit+ IPS panel with optical bonding (LOCA + UV cure), PCAP touch, anti-glare + anti-fingerprint glass. RAM Mount compatible.

---

## Related Documents

- [Wiring and Bus Architecture](wiring.md)
- [Connector Standard](connectors.md)
- [Bill of Materials](bom.md)
