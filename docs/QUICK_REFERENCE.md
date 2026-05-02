# Quick Reference Guide

## Purpose

Quickly understand the Polar Feeder system, runtime commands, and key file locations.

## Module Purpose Summary

| Module | Purpose | Key Classes/Functions |
|--------|---------|---------------------|
| `main.py` | App orchestrator | `main()`, BLE command handler, runtime state |
| `feeder_fsm.py` | LURE FSM logic | `FeederFSM`, `State` enum |
| `inverse_feeder_fsm.py` | INVERSE FSM logic | `InverseFeederFSM`, `InverseState` enum |
| `vision.py` | Motion detection and fusion | `VisionTracker`, `SensorFusion` |
| `radar.py` | Radar serial parsing | `RadarReader`, `RadarReading` |
| `ble_interface.py` | BLE UART server | `BleServer`, `BleCommand` |
| `actuator.py` | Actuator interface | `Actuator` |
| `transmittingfunc.py` | RF pulse transmission | `transmit1()`, `transmit2()` |
| `config/loader.py` | Config validation | `load_config()` |
| `logging/csv_logger.py` | CSV event logging | `CsvSessionLogger` |

## FSM State Overview

### LURE Mode

```
IDLE -> LURE -> RETRACT_WAIT -> COOLDOWN -> IDLE
```

### INVERSE Mode

```
IDLE -> WATCHING -> DISPENSING -> REWARDING -> COOLDOWN -> WATCHING
```

## BLE Command Quick Reference

### Enable/Disable

- `ENABLE=1` — Enable feeder
- `ENABLE=0` — Disable feeder

### Mode Selection

- `MODE=LURE` — Switch to LURE mode
- `MODE=INVERSE` — Switch to INVERSE mode

### Manual Actuation

- `ACTUATOR=EXTEND` — Extend actuator
- `ACTUATOR=RETRACT` — Retract actuator
- `RETRACT` — Manual FSM retract while in FEEDING/REWARDING

### Live Parameters

- `SET mt=<value>` — Set current mode motion threshold
- `SET motion_threshold=<value>` — Same as `mt`
- `SET rd=<value>` — Set `lure.retract_delay_ms`
- `SET retract_delay_ms=<value>` — Same as `rd`
- `SET sm=<value>` — Set `inverse.stillness_min_duration_s`
- `SET stillness_min_duration_s=<value>` — Same as `sm`
- `SET noise_buffer=<value>` — Set `inverse.noise_buffer_multiplier`
- `SET detect_dist=<value>` — Set `radar.detection_distance_m`
- `SET feed_dist=<value>` — Set `actuator.feeding_distance_m`
- `SET djump=<value>` — Set `radar.distance_jump_m`
- `SET pulse_ms=<value>` — Set RF pulse duration
- `SET telemetry_hz=<value>` — Set CSV telemetry frequency
- `SET radar_enabled=0|1` — Toggle radar
- `SET log_enabled=0|1` — Toggle logging
- `SET ble_disconnect_safe_idle=0|1` — Toggle safe idle on BLE disconnect

### Queries

- `GET <key>` — Read current runtime value
- `STATUS` — Get status snapshot
- `PING` — Heartbeat check

## Configuration Quick Reference

The active config sections are:

- `lure`
- `inverse`
- `logging`
- `radar`
- `safety`
- `actuator`
- `vision`

### Example

```json
{
  "lure": { "motion_threshold": 25.0, "retract_delay_ms": 0, "cooldown_s": 2.0 },
  "inverse": { "motion_threshold": 5.0, "stillness_min_duration_s": 1.0, "noise_buffer_multiplier": 1.0, "cooldown_s": 2.0 },
  "logging": { "enabled": true, "telemetry_hz": 5, "max_storage_mb": 500, "log_dir": "logs" },
  "radar": { "enabled": true, "port": "/dev/ttyRADAR", "baud": 115200, "timeout_s": 0.1, "zone_m": [1.0, 2.0, 3.0], "distance_jump_m": 0.05, "detection_distance_m": 3.0 },
  "safety": { "ble_disconnect_safe_idle": false },
  "actuator": { "pulse_ms": 200, "feeding_distance_m": 0.33 },
  "vision": { "enabled": true, "sync_window_s": 0.5 }
}
```

## Running the System

### BLE test mode

```bash
python src/pi/polar_feeder/main.py --ble-test --config config/config.example.json
```

### Demo mode

```bash
python src/pi/polar_feeder/main.py --config config/config.example.json --demo-seconds 60
```

### Custom config

```bash
python src/pi/polar_feeder/main.py --config config/my-config.json --ble-test
```

## File Locations

| Purpose | Location |
|---|---|
| Main code | `src/pi/polar_feeder/` |
| Config template | `config/config.example.json` |
| RF signals | `config/rf_signal1.json`, `config/rf_signal2.json` |
| Logs | `logs/session_*.csv` |
| Docs | `docs/*.md` |

## Common Issues & Fixes

- Feeder won't extend: Check `rf_signal1.json` and GPIO17 access
- BLE not working: Run with `--ble-test`
- Commands not responding: Ensure newline termination
- Radar not detecting threats: Lower `djump`
- Feeder keeps retracting: Increase `retract_delay_ms`
- High CPU usage: Reduce telemetry / loop frequency

## Branch Reference

- `main` — final submission code
- `develop` — active development branch
- `zoo-visit` — zoo deployment snapshot
- `archive/yolo-original` — legacy YOLO branch

---

**Save this file for quick reference during development!**
