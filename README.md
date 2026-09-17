# Autonomous Wildlife Enrichment System

Embedded vision, sensing, and real-time control system developed as an NC State ECE Senior Design project.

The system uses a Raspberry Pi 5 to combine camera-based object detection, 60 GHz radar sensing, finite-state-machine control, BLE telemetry/configuration, and RF actuator control to autonomously operate a wildlife enrichment device.

## System Highlights

- Raspberry Pi 5 running Embedded Linux
- 20 Hz main control loop across 5 concurrent threads
- 640x480 PiCamera2 image capture with YOLO/NCNN inference
- Infineon BGT60TR13C 60 GHz radar over 115200-baud UART
- Vision/radar sensor fusion and synchronization
- Finite state machines for autonomous operating modes
- Fault detection, communication timeouts, and fail-safe behavior
- Nordic UART Service BLE interface with 20 operator commands
- GPIO-based replay of a reverse-engineered RF actuator waveform
- 25-field CSV telemetry logging at 5 Hz
- JSON-schema-validated runtime configuration
- Linux systemd deployment and automated hardware self-test

## System Architecture

```text
                         +----------------------+
PiCamera2 -------------->|                      |
                         |   Vision Processing  |
YOLO / NCNN ------------>|                      |
                         +----------+-----------+
                                    |
                                    v
                             Vision Detection
                                    |
                                    |
BGT60TR13C Radar --UART--------------+------+
                                           |
                                           v
                                  +------------------+
                                  | Sensor Fusion /  |
                                  | Synchronization  |
                                  +--------+---------+
                                           |
                                           v
                                  +------------------+
Android App --BLE---------------->|   Control FSM    |
                                  | LURE / INVERSE   |
                                  +--------+---------+
                                           |
                                           v
                                  +------------------+
                                  | Safety / Fault   |
                                  | Handling         |
                                  +--------+---------+
                                           |
                                           v
Raspberry Pi GPIO ----------> RF Pulse Replay ----------> Linear Actuator

                     Telemetry / Events
                            |
                            v
                       CSV Logging
```

## Embedded Software Architecture

The application is organized into modular hardware-interface, control, sensing, and logging components:

| Module | Purpose |
| --- | --- |
| `main.py` | Application orchestration, threading, control loop, and runtime state |
| `vision.py` | Camera processing, object detection, and vision state |
| `radar.py` | UART radar interface, distance filtering, and threat detection |
| `feeder_fsm.py` | LURE operating-mode finite state machine |
| `inverse_feeder_fsm.py` | INVERSE operating-mode finite state machine |
| `actuator.py` | High-level actuator control and safety interface |
| `transmittingfunc.py` | Timing-sensitive GPIO RF waveform replay |
| `ble_interface.py` | Nordic UART Service BLE command/telemetry interface |
| `config/loader.py` | JSON configuration loading and schema validation |
| `logging/csv_logger.py` | Event and telemetry logging |

Primary source code is located under:

```text
src/pi/polar_feeder/
```

## Real-Time Control

The system executes a 20 Hz main control loop while sensor acquisition, computer vision, BLE communication, and logging operate across five concurrent threads.

Camera detections and radar measurements are synchronized within a bounded time window before being consumed by the active control state machine.

Two runtime-selectable operating modes are implemented:

### LURE

Deploys the enrichment actuator to attract the animal and retracts it when motion or proximity indicates a threat condition.

### INVERSE

Keeps the actuator retracted until the animal satisfies configured stillness and proximity conditions, then deploys the enrichment reward.

The active state machine can be changed at runtime through the BLE interface.

## Computer Vision

A PiCamera2 camera provides 640x480 image frames to a YOLO model optimized for NCNN inference on the Raspberry Pi.

The vision subsystem performs object detection and motion analysis before publishing perception state to the control system.

Vision information is combined with radar measurements rather than allowing either sensor to independently control the actuator.

## Radar Interface

An Infineon BGT60TR13C 60 GHz radar sensor provides ranging data over a 115200-baud UART connection.

The radar subsystem includes:

- serial parsing
- configurable detection zones
- distance filtering
- jump rejection
- read timeouts
- threat-state generation

Radar and vision data are synchronized before control decisions are made.

## RF Actuator Interface

The linear actuator's original RF control protocol was captured and reproduced in software.

The transmitter implementation replays a 174-edge GPIO waveform with:

- approximately 126 ms transmission envelope
- 191 us minimum pulse width
- direct GPIO timing control

This allows the Raspberry Pi to command the actuator without the original vendor remote.

## Fault Handling and Safety

The system includes multiple failure-handling mechanisms designed to place the actuator in a safe state when sensor or communication assumptions are violated.

Examples include:

- radar read timeout
- bounded vision/radar synchronization window
- BLE inactivity safe idle
- actuator interlocks
- manual-override suppression window
- configuration range validation
- graceful remote shutdown

Runtime configuration values are validated against a JSON schema before use.

## BLE Operator Interface

A Nordic UART Service GATT server provides remote configuration, telemetry, and manual control through an Android operator application.

The interface supports 20 commands, including:

```text
ENABLE=0 / ENABLE=1
MODE=LURE / MODE=INVERSE
SET <key>=<value>
GET <key>
STATUS
ACTUATOR=EXTEND
ACTUATOR=RETRACT
RETRACT
PING
SHUTDOWN
```

The BLE subsystem was hardened during field testing to handle bonding, device-address behavior, and serial-device enumeration issues.

## Telemetry and Testing

The system records 25-field CSV telemetry at 5 Hz for debugging, system verification, and parameter tuning.

Logged data includes:

- active FSM state
- camera/object-detection state
- radar distance and threat state
- fused threat state
- actuator/manual-override state
- runtime parameters
- commands and results
- fault codes

A scripted self-test is included under:

```text
tools/selftest.py
```

## Repository Structure

```text
polar-feeder/
|-- src/
|   `-- pi/
|       |-- polar_feeder/
|       `-- scripts/
|-- config/
|-- deploy/
|   `-- systemd/
|-- docs/
|-- tools/
|-- bluetooth/
|-- requirements.txt
`-- README.md
```

## Running on Raspberry Pi

Clone the repository:

```bash
git clone https://github.com/jlpena2002/polar-feeder-copy.git
cd polar-feeder-copy
```

Create a virtual environment:

```bash
python3 -m venv --system-site-packages .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Run the hardware/BLE configuration:

```bash
bash src/pi/scripts/run.sh --ble-test --config config/config.example.json
```

Run the self-test:

```bash
python tools/selftest.py
```

## Deployment

A systemd service is provided for unattended startup on the Raspberry Pi:

```bash
sudo cp deploy/systemd/polar-feeder.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable polar-feeder
sudo systemctl start polar-feeder
```

Monitor the service with:

```bash
sudo journalctl -u polar-feeder -f
```

## Documentation

Additional implementation and operating documentation is available under [`docs/`](docs/), including:

- Raspberry Pi setup
- BLE pairing
- runtime configuration
- manual operation
- codebase documentation
- testing references

## Project Context

This project was developed as an NC State Electrical and Computer Engineering Senior Design capstone.

The system was designed to demonstrate autonomous sensing and physical control in a real deployment environment, with emphasis on embedded software integration, system reliability, field debugging, and safe interaction between perception software and physical hardware.