# Connector Standard

## Micro-C (NMEA 2000 Compatible)

All HelmOS modules use the Micro-C connector, the same standard used by NMEA 2000. This choice provides:

- **IP67 waterproof rating** — fully sealed against splash and spray
- **Locking mechanism** — will not disconnect from vibration or movement
- **Marine-grade** — designed for the marine environment
- **Widely available** — stocked at marine chandleries worldwide
- **5-pin** — carries CAN data and 12V power in one connector

---

## Pinout

| Pin | Signal | Color (standard) | Description |
|-----|--------|-----------------|-------------|
| 1 | Shield | Bare/drain | Cable shield, chassis ground |
| 2 | NET-S | Red | 12V supply (max 3A per segment) |
| 3 | NET-C | Black | Power ground |
| 4 | NET-H | White | CAN High |
| 5 | NET-L | Blue | CAN Low |

This pinout is identical to the NMEA 2000 Micro-C standard.

---

## Identifying HelmOS vs NMEA 2000 Cables

Both buses use physically identical connectors. To avoid cross-connection:

- **HelmOS CAN backbone** — label with orange heat-shrink or orange cable tie at each end
- **NMEA 2000 backbone** — standard unmarked or blue labeling
- Document the routing clearly in the vessel installation log

---

## Components

| Part | Description | Source |
|------|-------------|--------|
| Micro-C male connector | Module-side connector | Maretron, Actisense, or equivalent |
| Micro-C female connector | Backbone T-piece side | Maretron, Actisense, or equivalent |
| T-piece connector | Backbone tap for each module | Standard NMEA 2000 accessory |
| Backbone cable | 5-conductor shielded, 24AWG | Standard NMEA 2000 cable |
| Terminator | 120Ω Micro-C terminator | Standard NMEA 2000 accessory |

Budget alternatives are available from Aliexpress and other suppliers. Verify IP67 rating before purchasing.

---

## Mating Cycles

Micro-C connectors are rated for approximately 500 mating cycles. For permanent backbone connections, use T-pieces and leave them connected. Only drop cables should be connected and disconnected regularly.
