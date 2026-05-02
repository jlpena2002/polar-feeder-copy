# 📚 Documentation Index

## Overview

This index is the central navigation page for Polar Feeder documentation.
The root `README.md` provides the project overview and quick start, while this file points to deeper developer, configuration, and deployment references.

## Recommended Reading Order

1. [QUICK_REFERENCE.md](QUICK_REFERENCE.md) — Fast system and command overview
2. [CONFIG_GUIDE.md](CONFIG_GUIDE.md) — Live configuration reference
3. [CODEBASE_DOCUMENTATION.md](CODEBASE_DOCUMENTATION.md) — Full architecture and module design
4. [RPI5_QUICK_START.md](RPI5_QUICK_START.md) — Raspberry Pi 5 setup and validation
5. [config/RF_SIGNALS_README.md](config/RF_SIGNALS_README.md) — RF signal recording and replay

## Branch Documentation

The repository uses these named branches:

- `main` — clean documented final submission code
- `develop` — active development branch going forward
- `zoo-visit` — snapshot of what was run at the zoo, kept for reference
- `archive/yolo-original` — archived original YOLO implementation and `yolo_detect.py`

## 📖 Documentation Files

### Core System Documentation
- **[CODEBASE_DOCUMENTATION.md](CODEBASE_DOCUMENTATION.md)**
  - System architecture overview
  - Module responsibilities and design decisions
  - Data flow examples and state machine descriptions
  - Safety and failure handling
- **[CONFIG_GUIDE.md](CONFIG_GUIDE.md)**
  - Detailed JSON configuration explanation
  - BLE live tuning keys and runtime mappings
  - Valid ranges, examples, and troubleshooting
- **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)**
  - Fast lookup table for commands, file locations, and test patterns
  - FSM state diagrams and runtime behavior
  - Common issues and workarounds

### Hardware and RF Documentation
- **[RPI5_QUICK_START.md](RPI5_QUICK_START.md)**
  - Raspberry Pi 5 setup and run commands
  - Dependency and validation checklist
  - BLE and actuator test guidance
- **[config/RF_SIGNALS_README.md](config/RF_SIGNALS_README.md)**
  - RF signal recording format
  - How to create and store RF waveform files
  - Practical notes for transmission and replay

### Status and Index
- **[DOCUMENTATION_SUMMARY.md](DOCUMENTATION_SUMMARY.md)**
  - Coverage summary and verification checklist
- **[README_DOCUMENTATION.md](README_DOCUMENTATION.md)**
  - High-level documentation completion summary

## 🔍 Source Code Documentation

Every Python module under `src/pi/polar_feeder/` includes module-level docstrings and method comments.

### Primary Modules
- `main.py` — orchestrator, BLE processing, camera thread, radar loop, and FSM control
- `feeder_fsm.py` — LURE mode finite state machine
- `inverse_feeder_fsm.py` — INVERSE mode finite state machine
- `vision.py` — object motion detection and sensor fusion
- `radar.py` — serial radar reader and threat parsing
- `ble_interface.py` — BLE Nordic UART server and command parser
- `actuator.py` — actuator interface, safe movement, and RF signaling
- `transmittingfunc.py` — RF pulse replay and waveform timing
- `config/loader.py` — JSON schema validation and typed config object
- `logging/csv_logger.py` — session event and telemetry logging

## 🎯 Common Use Cases

### New developer
1. Read [QUICK_REFERENCE.md](QUICK_REFERENCE.md)
2. Read [CODEBASE_DOCUMENTATION.md](CODEBASE_DOCUMENTATION.md)
3. Inspect module docstrings in `src/pi/polar_feeder/`
4. Run the demo mode for a safe introduction

### Configure the feeder
1. Read [CONFIG_GUIDE.md](CONFIG_GUIDE.md)
2. Edit `config/config.example.json`
3. Use BLE `SET` commands for live tuning

### Fix a bug
1. Search docs for the relevant module
2. Review inline comments in source code
3. Use [CODEBASE_DOCUMENTATION.md](CODEBASE_DOCUMENTATION.md) for architecture context

### Add a feature
1. Understand the architecture in [CODEBASE_DOCUMENTATION.md](CODEBASE_DOCUMENTATION.md)
2. Review module entry points and command handling
3. Reference [CONFIG_GUIDE.md](CONFIG_GUIDE.md) for config impact

### Deploy to Raspberry Pi 5
1. Follow [RPI5_QUICK_START.md](RPI5_QUICK_START.md)
2. Verify dependencies and permissions
3. Run BLE test mode and monitor logs

## 🔗 Cross-References

- Config keys → [CONFIG_GUIDE.md](CONFIG_GUIDE.md)
- BLE commands → [CODEBASE_DOCUMENTATION.md](CODEBASE_DOCUMENTATION.md)
- FSM behavior → [QUICK_REFERENCE.md](QUICK_REFERENCE.md)
- Pi 5 setup → [RPI5_QUICK_START.md](RPI5_QUICK_START.md)
- RF recording → [config/RF_SIGNALS_README.md](config/RF_SIGNALS_README.md)

## File Locations

- Source code: `src/pi/polar_feeder/`
- Config template: `config/config.example.json`
- RF signals: `config/rf_signal1.json`, `config/rf_signal2.json`
- Logs: `logs/*.csv`
- Systemd service file: `deploy/systemd/polar-feeder.service`
- Documentation: `docs/*.md`

## Notes

Keep this index updated when the repository structure or branch usage changes.
If documentation and code diverge, update both the markdown and the source docstrings together.
