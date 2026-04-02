# Configuration System

All HelmOS configuration lives in a single `config.json` file located at `~/helmos-core/config.json`. It is the single source of truth for the entire system.

---

## Full Schema

```json
{
  "vessel": {
    "name": "",
    "type": "motorboat",
    "length_m": 0,
    "engine_hp": 0,
    "max_speed_knots": 0
  },

  "rudder": {
    "max_degrees": 35,
    "zero_raw": 0,
    "port_raw": 0,
    "starboard_raw": 0,
    "invert": false
  },

  "pump": {
    "invert_direction": false,
    "max_speed_percent": 100
  },

  "autopilot": {
    "pid": {
      "Kp": 1.0,
      "Ki": 0.01,
      "Kd": 0.5
    },
    "dead_zone_degrees": 1.0,
    "max_rudder_speed_percent": 100,
    "rudder_speed_curve": "linear"
  },

  "sensors": {
    "gps_source": "nmea2000",
    "heading_source": "nmea2000"
  },

  "network": {
    "mqtt_host": "localhost",
    "mqtt_port": 1883,
    "api_port": 8000,
    "ui_port": 5173
  },

  "modules": {
    "autopilot-01": {
      "type": "autopilot",
      "configured": false,
      "last_seen": ""
    }
  },

  "system": {
    "first_run": true,
    "setup_complete": false,
    "config_version": "1.0"
  }
}
```

---

## First Run

When `system.first_run` is `true`, the UI displays the setup wizard at startup. The wizard collects vessel information and guides the user through hardware calibration. When complete, `first_run` is set to `false`.

---

## Rudder Calibration

The rudder angle sensor (AS5048A) returns a raw value between 0–65535. The calibration process maps this to a physical angle.

**Calibration wizard steps:**
1. Turn rudder to full port — save raw value as `port_raw`
2. Turn rudder to full starboard — save raw value as `starboard_raw`
3. Center rudder (straight ahead) — save raw value as `zero_raw`

The core calculates the normalized angle (-100 to +100) from these values at runtime:

```
physical_degrees = map(raw, port_raw, starboard_raw, -max_degrees, +max_degrees)
normalized       = (physical_degrees / max_degrees) * 100
```

---

## Rudder Angle Scale

The UI and all internal references use a normalized scale:

```
-100  ←────────────  0  ────────────→  +100
Port                Center            Starboard
```

The physical degree range (e.g. ±35°) is a vessel-specific setting and is abstracted away above the calibration layer.

---

## Module Config

When a new module is discovered, it is added to the `modules` object with `configured: false`. After setup wizard completion, `configured` is set to `true` and module-specific settings are stored under the module's ID.

The full config is pushed to the module via MQTT after any change:

```
helmos/modules/<module_id>/config → { config object }
```

---

## API Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/config` | Return full config |
| POST | `/config` | Update full config |
| GET | `/config/{section}` | Return one section |
| POST | `/config/{section}` | Update one section |
