# Snapmaker U1 Heat Soak

Klipper G-code macro to warm an enclosed **Snapmaker U1** before ABS/ASA printing, using the **Top Cover's internal recirculation/filtration fan** (not the separate `cavity_fan`).

> **v0.1.0 — Experimental, not yet hardware-tested.** Manually verify compatibility with your installed U1 firmware before automating print start.

## Features

- Bed target: 0–100 °C (default: 100 °C).
- Timed soak: 1–120 minutes (default: 30 minutes).
- Top Cover fan speed: 10–100% (default: 40%).
- Soak timer begins only after the bed reaches temperature.
- Reports final chamber temperature from `temperature_sensor cavity`.
- Top Cover internal recirculation via `SET_PURIFIER_MODE MODE=3`.

## Installation

1. Back up your existing printer configuration.
2. Copy [heat_soak.cfg](heat_soak.cfg) into the printer's Klipper configuration directory.
3. Add `[include heat_soak.cfg]` to `printer.cfg` if supported by your firmware.
4. Reload the configuration (typically `RESTART`).
5. Perform a short **attended** test with the Top Cover installed. Verify internal recirculation operates as expected.

The U1's configuration and ability to edit it may differ with firmware versions. The macro requires `SET_PURIFIER_MODE`, `RESPOND`, and the `cavity` temperature sensor.

## Usage

Standard ABS/ASA starting point:

```gcode
HEAT_SOAK BED=100 MINUTES=30 FAN=0.4
```

Brief attended test:

```gcode
HEAT_SOAK BED=60 MINUTES=2 FAN=0.4
```

The macro leaves the bed heated at completion so printing can begin. Call the macro manually first; only add it to your Snapmaker Orca print-start sequence once you have verified that any slicer-issued purifier commands do not override fan settings.

## Important limitations and safety

- This is a **fixed-time soak**, not an automatic chamber-target wait. The chamber temperature is only reported at the end.
- The 120-minute limit applies to the soak dwell only; the bed warm-up time (`M190`) is additional and can take longer.
- The `G4` dwell blocks the macro; it is not a separately cancellable background timer. Test the printer's normal cancellation behavior.
- At the end the macro sets idle timeout to 600 seconds, a common default. Change this to your printer's configured value if different.
- The exact behavior of `SET_PURIFIER_MODE` depends on your U1 firmware. Ensure fan circulation does not unintentionally exhaust the warm enclosure.
- Do not leave the machine unattended during initial testing. Follow appropriate ventilation and manufacturer precautions for ABS/ASA.
- The macro intentionally does not disable heating after the soak; monitor the printer.

## Compatibility references

- [Snapmaker official U1 Klipper configuration](https://github.com/Snapmaker/u1-klipper/blob/main/lava/printer.cfg) — documents the bed heater (100 °C max), `temperature_sensor cavity`, and `purifier` subsystem.
- Firmware behavior may change. Check the commands supported by your exact version.

## Roadmap

- Validate on real U1 hardware and specific firmware versions.
- Optional chamber-temperature threshold with timeout and stabilization period.
- Better cancellation, idle-timeout restoration, and slicer integration.

## License

[GNU GPL v3.0](LICENSE). Unofficial community project; not affiliated with or endorsed by Snapmaker.
