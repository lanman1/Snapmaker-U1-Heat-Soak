# Snapmaker U1 Heat Soak

Experimental Klipper heat soak for ABS/ASA on the **Snapmaker U1** using the **cavity/curtain air fan** (`cavity_fan`), not the Top Cover purifier.

**v0.3.0 — untested on physical hardware.** This version replaces blocking `M190` and `G4` with a cancelable 2-second `delayed_gcode` loop.

## Commands

| Command | Behavior |
| --- | --- |
| `HEAT_SOAK BED=100 MINUTES=20 FAN=0.4` | Begin heating, then soak 20 minutes after bed reaches target (within 1°C) |
| `HEAT_SOAK_STATUS` | Report phase, temperatures and approximate remaining time |
| `CANCEL_HEAT_SOAK` | Cancel and switch off bed and cavity fan |
| `CANCEL_HEAT_SOAK KEEP_BED=1` | Cancel and stop fan, **leave bed at existing target** |

**If the bed temperature is changed by another command while heat soak is active**, the macro automatically stops within approximately 2 seconds, switches off the cavity fan, and **preserves the newly chosen bed target**. To guarantee both heaters and the macro stop, use `CANCEL_HEAT_SOAK`; do not rely on temperature adjustment as a substitute for cancellation.

For example, during a soak:

```gcode
M140 S90
```

The macro detects the new 90°C setpoint and cancels itself without setting the bed back to 100°C. Conversely `M140 S0` sets the bed off, and the macro cancels shortly afterward.

## Installation

1. Back up printer configuration.
2. Copy [heat_soak.cfg](heat_soak.cfg) into your Klipper config directory.
3. Include it with `[include heat_soak.cfg]`, if firmware permits, and restart Klipper.
4. Confirm printer supports `[fan_generic cavity_fan]`, `[temperature_sensor cavity]`, `[delayed_gcode]`, and `RESPOND`.
5. Perform an attended short test:

```gcode
HEAT_SOAK BED=60 MINUTES=2 FAN=0.2
HEAT_SOAK_STATUS
CANCEL_HEAT_SOAK
```

## Parameters / behavior

- `BED`: 1–100°C, default 100°C. U1 published config specifies max bed temperature 100°C.
- `MINUTES`: 1–120 minutes, default 30. Soak begins within approximately 1°C of setpoint.
- `FAN`: 0–1, default 0.4.
- `HEAT_TIMEOUT`: 1–120 minutes, default 45. If bed does not reach target in this time, stops soak **and disables bed heater**.
- Completion leaves bed heater on at the selected target but stops cavity fan. It restores the configured idle timeout.
- Cancel default stops the bed heater and fan. `KEEP_BED=1` leaves the current bed target unchanged.
- The status remaining-time estimate is rounded and based on 2-second ticks; actual time may differ slightly under printer load.
- Changing the bed target by more than 0.5°C triggers automatic stop on the next tick, without overwriting that new target.
- Do not run when a print is active. Initial firmware compatibility and cancellation response are unverified.

## Safety

This macro is **not yet hardware-validated**. The cavity fan might cool the chamber depending on airflow; compare chamber temperature with and without it. Temperature changing isn't always an emergency stop; use printer's physical stop if required. A firmware restart, system fault, or unexpected interruption may leave heater/fan state different from what the macro expects. Never leave the printer unattended during initial tests. Follow manufacturer temperature limits and ABS/ASA ventilation practices.

## Sources

- [Snapmaker U1 Klipper configuration](https://github.com/Snapmaker/u1-klipper/blob/main/lava/printer.cfg)
- [Klipper: Command templates and delayed_gcode](https://www.klipper3d.org/Command_Templates.html)

## License

[GNU GPL v3.0](LICENSE). Unofficial community project.
