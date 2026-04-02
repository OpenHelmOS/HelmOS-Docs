# Autopilot Module

The autopilot module provides automatic heading control using a PID controller. It controls a reversible hydraulic pump via an ESP32 motor driver.

---

## Hardware

| Component | Model |
|-----------|-------|
| MCU | ESP32 |
| CAN transceiver | SN65HVD230 |
| Motor driver | TMC2209 |
| Rudder angle sensor | AS5048A (magnetic absolute encoder) |
| Hydraulic pump | Lowrance/Octopus PUMP-1 compatible reversible pump |

---

## Control Loop

```
[GPS heading] ──────────────────────────────────────┐
                                                     ▼
[Target heading] ──► [Heading Error] ──► [PID Controller] ──► [Pump speed/direction]
                           ▲                                           │
                           │                                           ▼
                    [Current heading] ◄── [Vessel dynamics] ◄── [Rudder moves]
                           ▲
                    [AS5048A sensor] ◄── [Rudder angle]
```

The PID controller computes a motor output from the heading error. The rudder angle sensor closes the loop and feeds back into the heading calculation.

---

## Heading Error Calculation

Heading is circular (0–360°). Simple subtraction breaks at the 0°/360° boundary. The correct formula finds the shortest path:

```python
error = (target - current + 180) % 360 - 180
```

This ensures a vessel on heading 355° targeting 005° turns 10° right, not 350° left.

---

## PID Controller

```python
class HeadingController:
    def __init__(self, Kp=1.0, Ki=0.01, Kd=0.5):
        self.Kp = Kp
        self.Ki = Ki
        self.Kd = Kd
        self.integral = 0
        self.last_error = 0

    def compute(self, target, current, dt):
        error = (target - current + 180) % 360 - 180

        self.integral += error * dt
        derivative = (error - self.last_error) / dt
        self.last_error = error

        output = (self.Kp * error +
                  self.Ki * self.integral +
                  self.Kd * derivative)

        return max(-100, min(100, output))
```

**Parameters:**
- **Kp** (proportional) — direct response to heading error. Too high → oscillation.
- **Ki** (integral) — corrects persistent offset. Too high → slow oscillation.
- **Kd** (derivative) — brakes before target, reduces overshoot.

---

## Motor Speed Curve

The PID output maps directly to pump speed (-100 to +100). Effective behavior with default PID:

```
Error > 20°   → near full speed
Error 5–20°   → proportional slowdown
Error < 5°    → slow fine adjustment
Error < 1°    → dead zone, motor stops
```

The dead zone (`dead_zone_degrees` in config) prevents the motor hunting around the target heading.

---

## Rudder Angle Scale

All rudder position values use a normalized scale regardless of the vessel's physical rudder range:

```
-100       -50        0        +50       +100
 │          │         │         │          │
Port max   Port     Center  Starboard  Starboard
                               max
```

Physical degrees are mapped to this scale during calibration. The `max_degrees` config value sets the physical limit (typically 30–35° for most vessels).

```python
RUDDER_MAX_DEG = 35  # from config

def degrees_to_normalized(degrees):
    return (degrees / RUDDER_MAX_DEG) * 100

def normalized_to_degrees(normalized):
    return (normalized / 100) * RUDDER_MAX_DEG
```

---

## Rudder Calibration

Calibration is performed via the setup wizard in the UI.

1. **Port limit** — turn rudder to full port, press OK → saves `port_raw`
2. **Starboard limit** — turn rudder to full starboard, press OK → saves `starboard_raw`
3. **Center** — center the rudder, press OK → saves `zero_raw`

The AS5048A sensor returns a 14-bit absolute value (0–16383). These raw values are stored in config and used for real-time angle calculation.

---

## MQTT Topics

| Topic | Direction | Description |
|-------|-----------|-------------|
| `helmos/autopilot/state` | Subscribe | `off` / `standby` / `active` |
| `helmos/autopilot/target` | Publish | Target heading (degrees 0–360) |
| `helmos/autopilot/enable` | Publish | Activate autopilot |
| `helmos/autopilot/disable` | Publish | Deactivate autopilot |
| `helmos/rudder/angle` | Subscribe | Current normalized angle (-100 to +100) |
| `helmos/pump/set` | Publish | Pump speed/direction (-100 to +100) |

---

## Safety

- If the module heartbeat is lost, the core immediately publishes `helmos/pump/set → 0` to stop the pump
- Physical rudder limits are enforced in firmware — the pump stops at `port_raw` and `starboard_raw` regardless of software command
- Autopilot can be disabled from the UI, from a physical button on the module, or by any MQTT client publishing to `helmos/autopilot/disable`
