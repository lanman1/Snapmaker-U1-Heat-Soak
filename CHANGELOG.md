# Changelog

## v0.4.0 — 2026-10-11

- Add optional chamber target (`CHAMBER`) and continuous stabilization period (`STABILIZE`).
- Add overall `MAX_TIME` limit including warmup.
- Add `HEAT_SOAK_PRINT_GATE` experimental print-start PAUSE/RESUME integration.
- Print gate holds submitted print after cancellation or timeout; never auto-releases failed soaks.
- Add `HEAT_SOAK_CLEAR` gate recovery command and expanded status reporting.
- Preserve cancel behavior, cavity fan control, and changed-bed-target detection.
- Update comprehensive installation, Snapmaker Orca start-code, and validation instructions.

**Hardware verification: pending. Pause/resume start-code behavior requires testing.**

## v0.3.0 — 2026-10-10

- Nonblocking delayed G-code heat-soak polling; cancel/status commands.

## v0.2.0 — 2026-10-10

- Switch to cavity/curtain fan control (`cavity_fan`).

## v0.1.0 — 2026-10-10

- Initial experimental timed soak using Top Cover purifier.
