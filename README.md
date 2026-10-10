# Snapmaker U1 Heat Soak

Klipper heat-soak macro for ABS/ASA on the **Snapmaker U1**, now using the **cavity/curtain air fan** (`cavity_fan`) instead of the optional Top Cover purifier.

> **v0.2.0 — experimental / not hardware-tested.** Verify actual fan behavior and firmware support before adding the macro to your print-start sequence. The auxiliary fan may cool the chamber rather than warm it depending on airflow.

## How it works

1. Start the cavity/curtain fan using `SET_FAN_SPEED FAN=cavity_fan SPEED=...`.
2. Heat the bed using `M190`, waiting until the setpoint is reached.
3. Hold the bed temperature for a configurable number of minutes using `G4`.
4. Stop the cavity fan and report final chamber temperature using `temperature_sensor cavity`.

The Top Cover recirculation/filtration fan is **not controlled** by this version. The heated bed remains on after the soak, so printing can follow.

## Installation

1. Back up the printer's Klipper configuration.
2. Copy [heat_soak.cfg](heat_soak.cfg) into the configuration directory.
3. Add `[include heat_soak.cfg]` to `printer.cfg` if your firmware allows includes, then reload the config.
4. Verify `[fan_generic cavity_fan]` and `[temperature_sensor cavity]` are present in the U1 firmware.
5. Run a short **attended** test, confirm fan direction and that temperature readings work.

## Usage

```gcode
HEAT_SOAK BED=100 MINUTES=20 FAN=0.4
```

This waits for a 100°C bed and then soaks for 20 minutes with the cavity fan at 40%, before switching that fan off.

Short test:

```gcode
HEAT_SOAK BED=60 MINUTES=2 FAN=0.2
```

Parameters: `BED` 0–100°C (default 100); `MINUTES` 1–120 (default 30); `FAN` 0.0–1.0 (default 0.4).

## Limitations / safety

- The published U1 bed heater config limits temperature to 100°C. **Do not bypass hardware/firmware temperature limits.**
- This macro runs a **fixed duration**; it does not wait for a specified chamber temperature.
- The 120-minute cap limits only the soak dwell. `M190` preheating time is extra.
- The `G4` dwell is blocking, not a cancellable background timer; test your usual stop/cancel sequence.
- The macro turns off `cavity_fan` at the end rather than restoring its previous speed. It resets idle timeout to 600 seconds; edit this if your printer has a different default.
- The U1's auxiliary fan may change enclosure airflow and cooling, so compare chamber temperatures during testing with it enabled and disabled.
- It leaves the heated bed active after finishing. Supervise initial tests and follow ventilation guidance for ABS/ASA.

## Upstream reference

[Snapmaker U1 Klipper configuration](https://github.com/Snapmaker/u1-klipper/blob/main/lava/printer.cfg) — check for `[fan_generic cavity_fan]`, `[temperature_sensor cavity]` and bed heater limits. Firmware variants may differ.

## Future improvements

- Hardware validation and firmware compatibility notes.
- Chamber temperature threshold and bounded timeout.
- Nonblocking cancellation and prior state restoration.
- Snapmaker Orca integration.

## License

[GPL-3.0](LICENSE). Unofficial community project, not endorsed by Snapmaker.
