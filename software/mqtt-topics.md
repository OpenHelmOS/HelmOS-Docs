# MQTT Topics

All HelmOS communication uses MQTT with Mosquitto running on the RPi at port 1883.

## Topic Conventions

```
helmos/<subsystem>/<measurement>
helmos/<subsystem>/<device>/<action>
helmos/modules/<module_id>/<type>
```

---

## Navigation

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/gps/speed` | Subscribe | Speed over ground (knots) |
| `helmos/gps/heading` | Subscribe | True heading (degrees) |
| `helmos/gps/lat` | Subscribe | Latitude |
| `helmos/gps/lon` | Subscribe | Longitude |
| `helmos/gps/cog` | Subscribe | Course over ground (degrees) |

---

## Rudder and Steering

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/rudder/angle` | Subscribe | Current rudder angle (-100 to +100) |
| `helmos/rudder/angle/raw` | Subscribe | Raw AS5048A sensor value |
| `helmos/rudder/set` | Publish | Target rudder position (-100 to +100) |

---

## Autopilot

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/autopilot/state` | Subscribe | `off` / `standby` / `active` |
| `helmos/autopilot/target` | Publish | Target heading (degrees) |
| `helmos/autopilot/enable` | Publish | Enable autopilot |
| `helmos/autopilot/disable` | Publish | Disable autopilot |
| `helmos/autopilot/pid` | Publish | Update PID values (JSON) |

---

## Engine

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/engine/rpm` | Subscribe | Engine RPM |
| `helmos/engine/coolant_temp` | Subscribe | Coolant temperature (°C) |
| `helmos/engine/hours` | Subscribe | Engine hours |

---

## Pump

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/pump/set` | Publish | Motor speed and direction (-100 to +100) |
| `helmos/pump/state` | Subscribe | Current pump state |

---

## Lights

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/lights/nav` | Publish | Navigation lights on/off |
| `helmos/lights/anchor` | Publish | Anchor light on/off |
| `helmos/lights/cabin` | Publish | Cabin lights on/off |
| `helmos/lights/state` | Subscribe | Current lights state (JSON) |

---

## Environment

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/environment/temp` | Subscribe | Air temperature (°C) |
| `helmos/environment/humidity` | Subscribe | Relative humidity (%) |
| `helmos/environment/bilge` | Subscribe | Bilge water detected (bool) |

---

## Module Management

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/modules/announce` | Subscribe | Module boot announcement (JSON) |
| `helmos/modules/<id>/heartbeat` | Subscribe | Module heartbeat (30s interval) |
| `helmos/modules/<id>/config` | Publish | Push config to module |
| `helmos/modules/<id>/update` | Publish | Trigger OTA firmware update |
| `helmos/modules/<id>/status` | Subscribe | Module status |

---

## Testing with mosquitto_pub / mosquitto_sub

```bash
# Listen to all topics
mosquitto_sub -h localhost -t "helmos/#" -v

# Simulate GPS heading
mosquitto_pub -h localhost -t "helmos/gps/heading" -m "245"

# Simulate rudder angle
mosquitto_pub -h localhost -t "helmos/rudder/angle" -m "42"

# Control LED (dev test)
mosquitto_pub -h localhost -t "helmos/led/set" -m "on"
```
