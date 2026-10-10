# Changelog

## v0.3.0 — 2026-10-10

- Replace blocking bed warmup and dwell with nonblocking delayed G-code checks.
- Add `CANCEL_HEAT_SOAK`, optional `KEEP_BED=1`, and `HEAT_SOAK_STATUS`.
- Automatically cancel without overriding the user's newly selected bed target.
- Add bed warmup timeout and protections against multiple simultaneous soaks.
- Restore configured idle timeout; stop cavity fan during cleanup.
- Experimental, hardware validation pending.

## v0.2.0 — 2026-10-10

- Switch fan control to cavity/curtain fan `cavity_fan`.
- Stop cavity fan when completed and update documentation.

## v0.1.0 — 2026-10-10

- Initial experimental timed soak using Top Cover internal circulation.
