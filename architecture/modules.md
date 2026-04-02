# Module System

## Overview

HelmOS modules are self-describing. When a new module is connected to the CAN bus, it announces itself to the core, provides its configuration schema, and the UI activates the relevant pages automatically. No manual configuration of the core or UI is required.

---

## Module Lifecycle

```
1. Module powered on
        ↓
2. ESP32 connects to WiFi / CAN bus
        ↓
3. Publishes announce message to helmos/modules/announce
        ↓
4. Core receives announce
        ↓
   Known module?
   ├── Yes → set status "online", restore saved config
   └── No  → add to config.json with defaults
              → UI shows "New module found" notification
              → User runs setup wizard
              → Config saved and pushed back to module
        ↓
5. Module subscribes to helmos/modules/<id>/config
6. Module sends heartbeat every 30s
7. Core monitors heartbeat — missing → status "offline" + UI warning
```

---

## Announce Message

Published to `helmos/modules/announce` at every boot:

```json
{
  "module_id": "autopilot-01",
  "type": "autopilot",
  "version": "1.0.0",
  "capabilities": ["rudder_control", "heading_hold"],
  "ui": {
    "nav_item": {
      "label": "Autopilot",
      "icon": "compass",
      "route": "/autopilot"
    },
    "dashboard_widget": true,
    "settings_section": true
  },
  "config_schema": {
    "rudder": {
      "max_degrees":        { "type": "float", "default": 35.0, "min": 10, "max": 45 },
      "invert":             { "type": "bool",  "default": false },
      "zero_raw":           { "type": "int",   "default": 0 },
      "port_raw":           { "type": "int",   "default": 0 },
      "starboard_raw":      { "type": "int",   "default": 0 }
    },
    "pump": {
      "invert_direction":   { "type": "bool",  "default": false },
      "max_speed_percent":  { "type": "int",   "default": 100, "min": 0, "max": 100 }
    },
    "pid": {
      "Kp":                 { "type": "float", "default": 1.0 },
      "Ki":                 { "type": "float", "default": 0.01 },
      "Kd":                 { "type": "float", "default": 0.5 }
    }
  }
}
```

---

## Config Push

After setup wizard completion, core pushes the saved config back to the module:

```
helmos/modules/autopilot-01/config → { full config object }
```

The module subscribes to this topic and applies values immediately. This means the module is stateless — it always gets its config from the core on boot.

---

## UI Integration

The UI contains pre-built pages for all supported module types. Pages are hidden unless the corresponding module is active in `config.json`.

```tsx
const navItems = [
  { path: "/",          label: "Dashboard", always: true },
  { path: "/autopilot", label: "Autopilot", module: "autopilot" },
  { path: "/lights",    label: "Lights",    module: "lights" },
  { path: "/settings",  label: "Settings",  always: true },
]
.filter(item => item.always || activeModules.includes(item.module))
```

When a new module type is developed after initial installation, a helmos-ui update is required to add the new page. Updates are pulled from GitHub via the settings panel.

---

## Heartbeat

Each module publishes a heartbeat every 30 seconds:

```
helmos/modules/autopilot-01/heartbeat → { "ts": 1712000000 }
```

If the core does not receive a heartbeat within 60 seconds, the module is marked offline and the UI displays a warning. This is a critical safety feature for autopilot operation.

---

## Supported Module Types

| Type | Capabilities |
|------|-------------|
| `autopilot` | Rudder control, heading hold, PID steering |
| `lights` | Navigation lights, cabin lights, anchor light |
| `environment` | Temperature, humidity, bilge water detection |
| `expansion` | 8-channel general purpose I/O |
