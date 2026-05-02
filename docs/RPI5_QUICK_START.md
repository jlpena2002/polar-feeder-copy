# Polar Feeder Raspberry Pi 5 Quick Start / Validation Guide

This guide describes Raspberry Pi 5 deployment and verification for the Polar Feeder.
It focuses on the run scripts, dependency checks, BLE validation, and branch references.

## Purpose

- Verify Raspberry Pi 5 software and hardware readiness
- Explain runtime launcher options
- Document `lgpio` and BLE dependencies
- Provide exact BLE and demo commands for verification
- Record the main branch naming conventions used by this repo

## Branches

Use these branches for repository expectations:

- `main` — clean documented final submission code
- `develop` — active development branch going forward
- `zoo-visit` — snapshot from the zoo deployment, kept for reference
- `archive/yolo-original` — archived original YOLO implementation and `yolo_detect.py`

## Run Script Variants

### `./run.sh`

- Location: repository root
- Use for quick local development and manual testing
- Activates `.venv` and runs `python -m polar_feeder.main`
- Good for interactive checks

### `src/pi/scripts/run.sh`

- Location: `src/pi/scripts/run.sh`
- Recommended for service deployment and more robust execution
- Uses strict shell error handling and a verified Python interpreter
- Recommended for systemd or unattended runs

### systemd service

- File: `deploy/systemd/polar-feeder.service`
- Wraps `src/pi/scripts/run.sh`
- Useful for boot startup and service recovery

## Required Dependencies

Install on Raspberry Pi 5:

- `python3`
- `python3-venv`
- `python3-pip`
- `python3-lgpio` or `pip install lgpio`
- `bluez`
- `libcamera` / `picamera2`
- `v4l-utils`
- `systemd`

## Base Setup

```bash
cd c:/Users/Jeremy/SD2025-36-Arctic-Project
python3 -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

> On PowerShell use `.venv\Scripts\Activate.ps1` instead of `activate`.

## Hardware Notes

- Radar is connected over UART at `115200` baud.
- Radar port is configured in `config/config.example.json` under `radar.port`.
- RF actuator uses `GPIO17`.
- BLE uses the Nordic UART Service via `bluezero`.

## Run the App

### BLE test mode

```bash
python src/pi/polar_feeder/main.py --ble-test --config config/config.example.json
```

### Demo mode

```bash
python src/pi/polar_feeder/main.py --config config/config.example.json --demo-seconds 60
```

### Robust launch

```bash
src/pi/scripts/run.sh --ble-test --config config/config.example.json
```

## Verify Configuration Loading

```bash
python - <<'PY'
from polar_feeder.config.loader import load_config
cfg = load_config('config/config.example.json')
print(cfg)
PY
```

## Self-Test Utility

```bash
python tools/selftest.py
```

## Inspect Logs

```bash
tail -f logs/*.csv
```

## BLE Command Validation

Use a BLE terminal or app while `--ble-test` is running:

- `ENABLE=1`
- `ENABLE=0`
- `MODE=LURE`
- `MODE=INVERSE`
- `SET rd=1500`
- `SET mt=25`
- `SET detect_dist=3.0`
- `STATUS`
- `ACTUATOR=RETRACT`

Confirm the device responds with `ACK` or valid status output.

## Radar and Vision Validation

1. Set `radar.enabled=true`.
2. Confirm the serial device exists for `radar.port`.
3. Confirm the camera is available and `vision.enabled=true`.
4. Start BLE test mode and verify radar and vision data appear in logs.

## Actuator Validation

1. Start BLE test mode.
2. Send `ACTUATOR=EXTEND`.
3. Send `ACTUATOR=RETRACT`.
4. Confirm `logs/*.csv` includes actuator events.

## Safety Validation

1. Set `safety.ble_disconnect_safe_idle=true`.
2. Start BLE test mode.
3. Stop BLE traffic and verify the system returns to safe idle after a timeout.

## Troubleshooting

- BLE not advertising: ensure Bluetooth is enabled and the device is in BLE test mode.
- `lgpio` import failing: install the package via apt or pip.
- Radar port missing: update `radar.port` to the correct device.
- Actuator not responding: verify RF wiring and GPIO access.
- Camera issues: install `libcamera` and camera drivers.

## Reference Docs

- Architecture and design: `CODEBASE_DOCUMENTATION.md`
- Configuration details: `CONFIG_GUIDE.md`
- Quick commands: `QUICK_REFERENCE.md`
- RF signal details: `config/RF_SIGNALS_README.md`
- Documentation coverage: `DOCUMENTATION_SUMMARY.md`
- Navigation index: `DOCUMENTATION_INDEX.md`

## Branch Reference

- `main` — final submission code
- `develop` — active development branch
- `zoo-visit` — zoo reference snapshot
- `archive/yolo-original` — legacy YOLO implementation

## Quick Shell Copy

```bash
cd c:/Users/Jeremy/SD2025-36-Arctic-Project
.venv\Scripts\activate
python src/pi/polar_feeder/main.py --ble-test --config config/config.example.json
# OR
src/pi/scripts/run.sh --ble-test --config config/config.example.json
# watch logs
tail -f logs/*.csv
# self-test
python tools/selftest.py
```
