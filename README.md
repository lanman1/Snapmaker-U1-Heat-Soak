# Snapmaker U1 Heat Soak

**v0.4.0 — Experimental. Not hardware-tested.** Klipper macro for ABS/ASA using the U1's **cavity/curtain fan** (`cavity_fan`), plus optional chamber-temperature soaking and a **print wait gate**.

## Install

Back up Klipper config, copy [heat_soak.cfg](heat_soak.cfg) to the Klipper configuration directory, include with `[include heat_soak.cfg]` and restart if the U1 firmware allows it. Requires `[fan_generic cavity_fan]`, `[temperature_sensor cavity]`, `[pause_resume]`, `[virtual_sdcard]`, and `RESPOND`. Not all U1 firmware or Snapmaker Orca printing workflows are known to support this integration; **verify on your device before using unattended**.

## Commands

```gcode
# Chamber mode: 40C continuously for 10 min, total timeout 60 min
HEAT_SOAK BED=100 CHAMBER=40 STABILIZE=10 MAX_TIME=60 FAN=0.4

# Timed mode: after bed is hot, soak 20 min (CHAMBER defaults to 0)
HEAT_SOAK BED=100 MINUTES=20 MAX_TIME=60 FAN=0.4

HEAT_SOAK_STATUS

# Cancel and shut off bed and fan (a waiting print stays paused)
CANCEL_HEAT_SOAK

# Cancel, keep current bed target (waiting print stays paused)
CANCEL_HEAT_SOAK KEEP_BED=1
```

Set `CHAMBER=0` (default) for timed mode; use `CHAMBER=20..50` for chamber mode. The optional `STABILIZE` parameter (default 10) requires a continuous period at or above the chamber threshold after the bed heats; it resets if the chamber drops below target. `MAX_TIME` is a hard macro deadline including bed warmup and soaking, default 60 minutes. This is a software macro timer, not independent thermal protection. The printer's native heater protections remain essential. A zero-minute stabilization accepts chamber target on the next two-second monitoring check.

## Submitting a print while soaking

**Experimental pause/resume integration requiring validation on the U1.** In Snapmaker Orca's printer-profile **Machine start G-code**, make the following the **FIRST executable line**, before bed/heater commands, homing, tool changes, or purge:

```gcode
HEAT_SOAK_PRINT_GATE
```

Then preserve all the existing Snapmaker U1 start G-code *after* this line. Do not replace the U1's standard print-start sequence. In theory, Klipper's `PAUSE` prevents virtual-SD print execution from continuing while the manual soak runs. Once it completes successfully, a delayed handler issues `RESUME` only if the printer still reports a paused job. **However, the U1's OEM pause/resume macros and slicer submission path may differ. This has not been tested.** If the U1 does not actually pause before executing subsequent slicer startup commands, **do not use the print gate**.

If no heat soak is active, the gate does nothing and normal printing proceeds.

### Cancellation and failure during a waiting print

- `CANCEL_HEAT_SOAK` turns off the bed and cavity fan and disables automatic resume. Any waiting print **remains held**.
- `CANCEL_HEAT_SOAK KEEP_BED=1` preserves bed target but **does not** release the print.
- Changing the bed target by more than 0.5°C automatically cancels the soak on the next scheduled check and preserves the newly selected target.
- Missing the `MAX_TIME` deadline cancels the soak, turns off bed and fan, and leaves waiting print paused.
- **To dispose of a held print, use the U1's normal Cancel Print control.** Once the job is no longer printing or paused, run `HEAT_SOAK_CLEAR` to reset the gate for future prints.
- Do **not** press Resume on a held print after a failed/canceled soak unless you have separately confirmed it is safe to proceed.
- Do **not** use `HEAT_SOAK_CLEAR` while the print remains paused; it refuses.
- Klipper/emergency shutdown, service restart, or manual operations may bypass macro state management. Always check machine state.

## Recommended supervised validation

1. Install and reload the macro.
2. Run `HEAT_SOAK BED=60 MINUTES=3 FAN=0.2 MAX_TIME=15` and verify fan direction and sensor status.
3. Submit a **small dummy print** with `HEAT_SOAK_PRINT_GATE` as the first line of machine start G-code and verify that *no* later start commands execute before soak completes.
4. Verify it resumes only at successful completion. Repeat and test cancellation; confirm cancellation does **not** restart the job.
5. Test a chamber target (for example `CHAMBER=30 STABILIZE=2 MAX_TIME=20`) while physically supervising.
6. Confirm slicer bed temperature commands after gate release do not inadvertently reset printing parameters. Snapmaker OEM behavior can differ from stock Klipper.

## Known limitations

- Not validated on actual hardware. Printing from non-virtual-SD streams may not follow `PAUSE` semantics.
- Automatic release assumes `printer.pause_resume.is_paused` and `printer.print_stats.state` are compatible with the U1's firmware.
- The macro does **not** implement a true queued-job system; it only gates a single submitted job.
- The bed remains heated after successful soak, and the cavity fan stops. Fan may cool the chamber depending on airflow.
- A manual print Resume can bypass the gate; do not resume a held job prematurely.
- Macro checks run approximately every two seconds; exact timing depends on host scheduling.
- U1 documented heater limit is 100°C. Do not override thermal safety limits.
- Follow ABS/ASA ventilation practices; do not leave the printer unattended during initial tests.

## Upstream

- [Klipper delayed G-code and macro templating](https://www.klipper3d.org/Command_Templates.html)
- [Klipper G-code and pause/resume](https://www.klipper3d.org/G-Codes.html)
- [Snapmaker U1 firmware configuration](https://github.com/Snapmaker/u1-klipper/blob/main/lava/printer.cfg)

## License

[GNU GPL v3.0](LICENSE). Community project, not endorsed by Snapmaker.
