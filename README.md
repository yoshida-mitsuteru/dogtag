# DogTag

A compact, implementation-oriented reference for a low-power GNSS/Sigfox
wearable location tracker, from development-board integration and embedded
firmware to power budgeting and system verification.

The initial target is autonomous GNSS position reporting over Sigfox every
15 minutes. The intended application is to help caregivers locate children
or older adults who have become lost.

## Scope

The material will cover:

- GNSS acquisition, fix validation, and UART interfacing
- Sigfox uplink payloads and cloud reception
- Low-power MCU operation and power regulation
- Battery and charging-circuit requirements
- Handling unavailable GNSS fixes and stale positions
- Power measurement and seven-day runtime evaluation
- System verification under different reception conditions

The emphasis is on the requirements, specifications, calculations, and
verification behind each implementation decision. The material is organized
as an educational engineering reference.

## Evaluation Hardware

| Function | Initial selection |
| --- | --- |
| MCU board | [ST NUCLEO-L031K6](https://www.st.com/en/evaluation-tools/nucleo-l031k6.html) (STM32L031K6) |
| GNSS | Quectel L76K |
| Sigfox | BRKLSM100A |
| Regulator | MCP1700, 3.0 V option |
| Battery | Single-cell LiPo, nominal 110 mAh target |

The implementation uses development boards and breadboard connections.
Operating performance and interface compatibility remain subject to testing.
GNSS reception and Sigfox coverage are not guaranteed, particularly in
obstructed terrain; this is not a continuous or guaranteed emergency service.

## Repository Status

Implementation and bench verification are in progress.

[Design notes](docs/design-notes.md), firmware, schematics, and measured
15-minute-cycle power results will be added or updated as they are verified.
