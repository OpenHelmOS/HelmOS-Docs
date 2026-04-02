# Lights Module

The lights module controls vessel navigation lights, anchor light, and cabin lighting via relay or MOSFET outputs on an ESP32.

---

## Hardware

| Component | Model |
|-----------|-------|
| MCU | ESP32 |
| CAN transceiver | SN65HVD230 |
| Output drivers | MOSFET or relay board (4–8 channels) |

---

## Supported Light Types

| Light | Description | COLREGS requirement |
|-------|-------------|---------------------|
| Navigation lights | Port (red), starboard (green), stern (white) | Required underway |
| Masthead light | White forward-facing | Required for powerboats |
| Anchor light | All-round white | Required at anchor |
| Cabin lights | Interior lighting | Optional |

---

## MQTT Topics

| Topic | Payload | Description |
|-------|---------|-------------|
| `helmos/lights/nav` | `on` / `off` | Navigation lights |
| `helmos/lights/anchor` | `on` / `off` | Anchor light |
| `helmos/lights/masthead` | `on` / `off` | Masthead light |
| `helmos/lights/cabin` | `on` / `off` | Cabin lights |
| `helmos/lights/state` | JSON | Current state of all lights |

---

## Config Schema (from announce message)

```json
{
  "channels": {
    "nav":      { "type": "int", "default": 0, "description": "GPIO pin for nav lights" },
    "anchor":   { "type": "int", "default": 1, "description": "GPIO pin for anchor light" },
    "masthead": { "type": "int", "default": 2, "description": "GPIO pin for masthead" },
    "cabin":    { "type": "int", "default": 3, "description": "GPIO pin for cabin" }
  },
  "invert_outputs": { "type": "bool", "default": false }
}
```

`invert_outputs` supports relay boards that are active-low.
