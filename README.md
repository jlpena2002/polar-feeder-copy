# Polar Feeder (Pi) Project

## Overview

Polar Feeder is an automated polar bear feeder system built for Raspberry Pi 5.
It combines radar sensing, computer vision, RF actuator control, and BLE-based
remote operator commands to safely deploy food while reacting to animal motion.

## Hardware

- Raspberry Pi 5 — main controller
- PSoC 6 BGT60TR13C radar board — distance sensing via UART at 115200 baud on `/dev/ttyACM0`
- PiCamera2 — computer vision input at `640x480`
- YOLOv8 NCNN model (`yolo26n_ncnn_model`) — polar bear detection (COCO class `21`)
- RF linear actuator — food arm controlled by replaying GPIO pulse waveforms on `GPIO17`
- Android app (MIT App Inventor) — BLE operator interface

## Software architecture

- `src/pi/polar_feeder/main.py` — entry point and orchestrator
- `src/pi/polar_feeder/feeder_fsm.py` — `LURE` mode finite state machine
- `src/pi/polar_feeder/inverse_feeder_fsm.py` — `INVERSE` mode finite state machine
- `src/pi/polar_feeder/vision.py` — vision tracking and sensor fusion
- `src/pi/polar_feeder/radar.py` — radar serial parser and threat detector
- `src/pi/polar_feeder/ble_interface.py` — Nordic UART Service BLE server
- `src/pi/polar_feeder/actuator.py` — high-level actuator control API
- `src/pi/polar_feeder/transmittingfunc.py` — RF pulse replay engine
- `src/pi/polar_feeder/config/loader.py` — JSON config loader and validator
- `src/pi/polar_feeder/logging/csv_logger.py` — session event and telemetry logging

## Modes of operation

- **BLE mode**: full hardware operation with remote BLE commands.
- **Demo mode**: simulated telemetry output without actuator or BLE.

## FSM modes

- **LURE** — extends food to attract the bear, retracts on threat, locks into `FEEDING` when the bear reaches `feeding_distance_m`.
- **INVERSE** — keeps the food retracted, rewards bear stillness by extending food, locks into `REWARDING` after stillness and close proximity.

## Branch guidance

This repository uses named branches for version control and reference snapshots:

- `main` — clean documented final submission code
- `develop` — active development branch going forward
- `zoo-visit` — snapshot of what was run at the zoo, kept for reference
- `archive/yolo-original` — archived original YOLO + `yolo_detect.py` versions

## BLE command reference

- `ENABLE=0` / `ENABLE=1` — stop/start feeder and camera thread
- `MODE=LURE` / `MODE=INVERSE` — switch active FSM mode
- `SET <key>=<value>` — update live runtime parameters (not persisted)
  - supported keys: `mt`, `motion_threshold`, `rd`, `retract_delay_ms`, `sm`, `stillness_min_duration_s`, `noise_buffer`, `detect_dist`, `feed_dist`, `djump`, `pulse_ms`, `telemetry_hz`, `radar_enabled`, `log_enabled`, `ble_disconnect_safe_idle`
- `GET <key>` — read a runtime value
- `STATUS` — return a full runtime snapshot
- `ACTUATOR=EXTEND` / `ACTUATOR=RETRACT` — manual actuator control and override
- `RETRACT` — invoke FSM manual retract if in `FEEDING` or `REWARDING`
- `PING` / `PONG` — heartbeat
- `SHUTDOWN` — graceful Pi shutdown

### Notes

- `SET` updates runtime state only and does not persist to config.
- `MODE` hot-swaps the FSM and restarts the camera if enabled.
- Manual actuator commands suppress FSM ticks for 5 seconds.
- BLE safe idle can force `ENABLE=0` after 30 seconds of BLE inactivity.

## Config reference

The JSON config contains these sections:

- `lure`
  - `motion_threshold` — base motion threshold in pixels
  - `retract_delay_ms` — delay before retracting after threat
  - `cooldown_s` — cooldown before next cycle
- `inverse`
  - `motion_threshold` — stillness threshold in pixels
  - `stillness_min_duration_s` — required stillness duration
  - `noise_buffer_multiplier` — buffer multiplier to prevent jitter resets
  - `cooldown_s` — cooldown after retraction
- `logging`
  - `enabled`, `telemetry_hz`, `max_storage_mb`, `log_dir`
- `radar`
  - `enabled`, `port`, `baud`, `timeout_s`, `zone_m`, `distance_jump_m`, `detection_distance_m`
- `safety`
  - `ble_disconnect_safe_idle`
- `actuator`
  - `pulse_ms`, `feeding_distance_m`
- `vision`
  - `enabled`, `sync_window_s`

## Logging

CSV session logs include event and telemetry rows with fields such as:
`timestamp_utc`, `session_id`, `test_id`, `event_type`, `state`, `fsm_mode`, `enable_flag`, `frame_index`, `obj_count`, `bear_detected`, `vision_motion`, `vision_threat`, `camera_active`, `detection_time_s`, `radar_dist_m`, `radar_threat`, `radar_enabled`, `radar_zone`, `fused_threat`, `motion_threshold`, `noise_buffer_multiplier`, `center_motion`, `size_change`, `fsm_substate`, `manual_override_active`, `command`, `result`, `fault_code`, and `notes`.

## Setup

```bash
cd c:/Users/Jeremy/SD2025-36-Arctic-Project
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Run BLE mode:

```bash
python src/pi/polar_feeder/main.py --ble-test --config config/config.example.json
```

Run demo mode:

```bash
python src/pi/polar_feeder/main.py --config config/config.example.json
```

Verify logs:

```bash
type logs\*.csv
```

Run self-test:

```bash
python tools/selftest.py
```

## Deployment

```bash
sudo cp deploy/systemd/polar-feeder.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable polar-feeder
sudo systemctl start polar-feeder
sudo journalctl -u polar-feeder -f
```

## Branch guidance

- `main` — stable release-ready code
- `develop` — integration/testing branch
- `feature/*` — work-in-progress development

Use `git branch -a` to inspect actual repository branches.

## Notes

- Active runtime values are not persisted unless written back to a config file manually.
- The BLE operator app communicates using the Nordic UART Service protocol.
- Detailed configuration and operational guidance is available in `docs/`.

