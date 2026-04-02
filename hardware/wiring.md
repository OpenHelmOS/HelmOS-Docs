# Wiring and Bus Architecture

## Two Separate CAN Buses

HelmOS operates two physically and logically separate CAN buses. They use the same connector standard but must never be bridged.

```
┌─────────────────────────────────────────────────────┐
│                  Raspberry Pi 5                     │
│                                                     │
│  MCP2515 #1 (CE0/GPIO25)    MCP2515 #2 (CE1/GPIO24)│
└──────────┬──────────────────────────┬───────────────┘
           │                          │
    HelmOS CAN                   NMEA 2000
    500 kbit/s                   250 kbit/s
           │                          │
    ┌──────┴──────┐            ┌──────┴──────┐
    │ ESP32       │            │ Lowrance    │
    │ Autopilot   │            │ Elite 7 Ti  │
    ├─────────────┤            └─────────────┘
    │ ESP32       │
    │ Lights      │
    ├─────────────┤
    │ ESP32       │
    │ Environment │
    └─────────────┘
    120Ω terminator             120Ω terminator
```

### HelmOS CAN (500 kbit/s)
- Carries HelmOS-native protocol between RPi and ESP32 modules
- All OpenHelmOS modules connect here
- Bridged to MQTT by helmos-core

### NMEA 2000 (250 kbit/s)
- Connects to existing commercial marine devices (chartplotter, VHF, engine gateway)
- Bridged to Signal K or canboat for data extraction
- HelmOS reads data from this bus but does not control devices on it

---

## Bus Topology

Both buses use a backbone + drop cable topology, identical to standard NMEA 2000 installations.

```
Backbone ────┬────────┬────────┬──── 120Ω
             │        │        │
           Drop     Drop     Drop
             │        │        │
          Module   Module   Module
```

- Backbone: shielded twisted pair, 5-conductor
- Drop cables: maximum 6 meters per drop
- Termination: 120Ω resistors at both ends of backbone
- Maximum backbone length: 100 meters

---

## Power Distribution

Modules are powered directly from the CAN backbone (NET-S / NET-C pins).

```
12V battery → fuse → backbone NET-S (pin 2)
GND         ────── → backbone NET-C (pin 3)
```

Each module draws power from the connector — no separate power wiring needed.

Recommended fuse: 3A per backbone segment.
