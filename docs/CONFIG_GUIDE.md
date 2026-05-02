# Configuration Guide for Polar Feeder

## Overview

The Polar Feeder uses JSON configuration to tune sensing, actuator timing, BLE safety, and logging behavior.
This guide documents the active config sections and the live BLE parameter mapping.

## Configuration File Structure

The configuration is organized into seven sections:

- `lure`
- `inverse`
- `logging`
- `radar`
- `safety`
- `actuator`
- `vision`

The canonical template is `config/config.example.json`.

## Section: `lure`

Controls LURE mode behavior.

```json
"lure": {
  "motion_threshold": 25.0,
  "retract_delay_ms": 0,
  "cooldown_s": 2.0
}
```

- `motion_threshold` — Vision motion threshold for LURE mode.
- `retract_delay_ms` — Delay in milliseconds before retracting after a threat.
- `cooldown_s` — Seconds to wait after retraction before starting a new LURE cycle.

**Typical ranges:**
- `motion_threshold`: 5.0 - 200.0
- `retract_delay_ms`: 0 - 3000
- `cooldown_s`: 0.0 - 30.0

## Section: `inverse`

Controls INVERSE mode behavior.

```json
"inverse": {
  "motion_threshold": 5.0,
  "stillness_min_duration_s": 1.0,
  "noise_buffer_multiplier": 1.0,
  "cooldown_s": 2.0
}
```

- `motion_threshold` — Maximum motion in pixels considered still.
- `stillness_min_duration_s` — Time in seconds required for stillness.
- `noise_buffer_multiplier` — Multiplier applied to the noise threshold during dispensing.
- `cooldown_s` — Seconds to wait after retraction before resuming WATCHING.

**Typical ranges:**
- `motion_threshold`: 5.0 - 200.0
- `stillness_min_duration_s`: 0.5 - 10.0
- `noise_buffer_multiplier`: 1.0 - 3.0
- `cooldown_s`: 0.0 - 30.0

## Section: `logging`

Controls CSV session logging.

```json
"logging": {
  "enabled": true,
  "telemetry_hz": 5,
  "max_storage_mb": 500,
  "log_dir": "logs"
}
```

- `enabled` — Enable or disable CSV logging.
- `telemetry_hz` — Frequency of telemetry writes.
- `max_storage_mb` — Storage limit for log files before rollover.
- `log_dir` — Directory to save logs.

**Typical ranges:**
- `telemetry_hz`: 1 - 30
- `max_storage_mb`: 10 - 5000

## Section: `radar`

Controls radar sensing and threat detection.

```json
"radar": {
  "enabled": true,
  "port": "/dev/ttyRADAR",
  "baud": 115200,
  "timeout_s": 0.1,
  "zone_m": [1.0, 2.0, 3.0],
  "distance_jump_m": 0.05,
  "detection_distance_m": 3.0
}
```

- `enabled` — Enable or disable radar input.
- `port` — Serial device port used by the radar.
- `baud` — Serial baud rate.
- `timeout_s` — Read timeout in seconds.
- `zone_m` — Distance zone metadata for the radar.
- `distance_jump_m` — Sudden distance change threshold for threat detection.
- `detection_distance_m` — Distance within which radar threat detection becomes active.

**Typical ranges:**
- `distance_jump_m`: 0.05 - 2.0 meters
- `detection_distance_m`: 0.5 - 50.0 meters

## Section: `safety`

Controls BLE disconnect safety behavior.

```json
"safety": {
  "ble_disconnect_safe_idle": false
}
```

- `ble_disconnect_safe_idle` — When true, the feeder is forced to safe idle if BLE disconnects.

## Section: `actuator`

Controls actuator signal timing and feeding distance.

```json
"actuator": {
  "pulse_ms": 200,
  "feeding_distance_m": 0.33
}
```

- `pulse_ms` — RF pulse duration in milliseconds.
- `feeding_distance_m` — Distance considered close enough to feed.

**Typical ranges:**
- `pulse_ms`: 50 - 1000
- `feeding_distance_m`: 0.1 - 5.0

## Section: `vision`

Controls computer vision data processing.

```json
"vision": {
  "enabled": true,
  "sync_window_s": 0.5
}
```

- `enabled` — Enable or disable vision processing.
- `sync_window_s` — Time window for radar/vision fusion alignment.

**Typical ranges:**
- `sync_window_s`: 0.1 - 5.0

## Live BLE Parameter Mapping

Some parameters are adjustable via BLE at runtime. These values are not written back to disk.

| BLE Key | Config Key | Notes |
|---|---|---|
| `mt` / `motion_threshold` | `lure.motion_threshold` or `inverse.motion_threshold` | Active mode only |
| `rd` / `retract_delay_ms` | `lure.retract_delay_ms` | LURE only |
| `sm` / `stillness_min_duration_s` | `inverse.stillness_min_duration_s` | INVERSE only |
| `noise_buffer` | `inverse.noise_buffer_multiplier` | INVERSE only |
| `detect_dist` | `radar.detection_distance_m` | All modes |
| `feed_dist` | `actuator.feeding_distance_m` | All modes |
| `djump` | `radar.distance_jump_m` | All modes |
| `pulse_ms` | `actuator.pulse_ms` | All modes |
| `telemetry_hz` | `logging.telemetry_hz` | All modes |
| `radar_enabled` | `radar.enabled` | All modes |
| `log_enabled` | `logging.enabled` | All modes |
| `ble_disconnect_safe_idle` | `safety.ble_disconnect_safe_idle` | All modes |

## Example Config Snippet

```json
{
  "lure": {
    "motion_threshold": 25.0,
    "retract_delay_ms": 0,
    "cooldown_s": 2.0
  },
  "inverse": {
    "motion_threshold": 5.0,
    "stillness_min_duration_s": 1.0,
    "noise_buffer_multiplier": 1.0,
    "cooldown_s": 2.0
  },
  "logging": {
    "enabled": true,
    "telemetry_hz": 5,
    "max_storage_mb": 500,
    "log_dir": "logs"
  },
  "radar": {
    "enabled": true,
    "port": "/dev/ttyRADAR",
    "baud": 115200,
    "timeout_s": 0.1,
    "zone_m": [1.0, 2.0, 3.0],
    "distance_jump_m": 0.05,
    "detection_distance_m": 3.0
  },
  "safety": {
    "ble_disconnect_safe_idle": false
  },
  "actuator": {
    "pulse_ms": 200,
    "feeding_distance_m": 0.33
  },
  "vision": {
    "enabled": true,
    "sync_window_s": 0.5
  }
}
```

## Validation

The config is validated against `schema.json` and loaded by `src/pi/polar_feeder/config/loader.py`.

## Loading Configuration

```python
from polar_feeder.config.loader import load_config
cfg = load_config('config/config.example.json')
print(cfg)
```

## Troubleshooting

- **Feeder does not trigger:** Lower `lure.motion_threshold`, reduce `inverse.stillness_min_duration_s`, or confirm radar input.
- **Feeder triggers too easily:** Increase `lure.motion_threshold` or `inverse.stillness_min_duration_s`.
- **Actuator behavior is wrong:** Adjust `lure.retract_delay_ms`, `actuator.pulse_ms`, and `actuator.feeding_distance_m`.
- **Radar threats fail:** Confirm `radar.enabled=true`, correct `radar.port`, and a valid serial device.
- **Logging missing:** Confirm `logging.enabled=true` and `log_dir` exists.

## Notes

- `config/config.example.json` is the repository template.
- Live BLE `SET` changes are runtime-only unless the JSON file is edited manually.
