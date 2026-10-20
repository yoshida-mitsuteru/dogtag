# DogTag Design Notes

This document describes the initial 15-minute reporting configuration.
The calculations below define engineering targets, not measured results.

## 1. Positioning and Communication

GNSS supplies absolute position without requiring nearby Wi-Fi access points
or cellular positioning infrastructure. Sigfox provides low-volume, wide-area
uplink through an existing network.

The two functions are independent: a valid GNSS fix does not prove that a
report reached the server, and a received Sigfox message does not guarantee
a fresh GNSS fix.

The Sigfox uplink payload limit is **12 bytes**. A 15-minute reporting period
requires

```text
24 h × 60 min / 15 min = 96 uplinks/day
```

The Japanese Sigfox Buy service offers daily uplink levels of 2, 70, and 140
messages. The **140-message/day** level accommodates the nominal 96 reports,
subject to any additional alerts and transmission attempts.

Reference: [KCCS Sigfox Buy](https://www.kccs.co.jp/sigfox/buy/).

## 2. Component Selection

| Component | Selection rationale | To verify |
| --- | --- | --- |
| [NUCLEO-L031K6](https://www.st.com/en/evaluation-tools/nucleo-l031k6.html) | Nucleo-32 connectors and on-board ST-LINK simplify firmware and breadboard evaluation. The STM32L031K6 provides USART and LPUART interfaces. | Both UART connections, pin mapping, ST-LINK connections, MCU sleep behavior |
| Quectel L76K | Standalone GNSS positioning for outdoor use, including locations without terrestrial positioning infrastructure. | Acquisition time, current, antenna orientation, accuracy under canopy or terrain obstruction |
| BRKLSM100A | Sigfox module for compact, infrequent position uplinks over a wide-area service. | Japan-compatible configuration, antenna, successful cloud receipt |
| MCP1700, 3.0 V option | Low-quiescent-current linear regulation with a simple supply arrangement. | Dropout, peak-load response, capacitor requirements |
| Nominal 110 mAh LiPo | Battery capacity target associated with a compact wearable form factor. | Usable capacity, discharge peaks, cutoff, protection, and permitted charging current |

Relevant device documentation:
[STM32L031K6 datasheet](https://www.st.com/resource/en/datasheet/stm32l031k6.pdf),
[NUCLEO-L031K6 user manual](https://www.st.com/resource/en/user_manual/dm00231744-stm32-nucleo32-boards-mb1180-stmicroelectronics.pdf),
and [Quectel L76K hardware design](https://files.waveshare.com/upload/d/db/Quectel_L76K_Hardware_Design_V1.0.pdf).

## 3. Power Budget

The battery-life target is **seven days** using a nominal **110 mAh** cell.
Seven days correspond to 168 hours, so the ideal average-current limit is

```text
I_avg = 110 mAh / 168 h = 0.655 mA
```

At 15-minute intervals there are 672 scheduled reporting cycles in seven
days. The corresponding ideal capacity allowance is

```text
Q_cycle = 110 mAh / 672 = 0.164 mAh per report
```

These are **upper bounds based on nominal capacity**. Actual usable battery
capacity is reduced by cutoff voltage, operating conditions, aging, and
design reserve.

The measured cycle energy must account for GNSS acquisition, Sigfox
transmission, MCU execution, and the remaining sleep period. When measuring
with a NUCLEO-L031K6 board, separate MCU and external-module consumption
from the on-board ST-LINK and other evaluation-board loads.

## 4. Voltage Regulation and Charging

The MCP1700 3.0 V option must be checked for output stability, dropout, and
current transients at the selected battery voltage. The manufacturer
specifies a **minimum 1.0 µF ceramic output capacitor** under the applicable
conditions.

Reference: [Microchip MCP1700 datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/MCP1700-Data-Sheet-20001826F.pdf).

The charging evaluation uses the [Adafruit BQ25185 charger board
(product 6091)](https://www.adafruit.com/product/6091). Its default
**500 mA** charge setting is not suitable for the nominal **110 mAh**
cell without verifying the cell's permitted charge current.

The design target is approximately **22 mA**, or **0.20 C** relative to
the nominal capacity. The BQ25185 sets fast-charge current through the
ISET-to-ground resistance:

```text
I_CHG = K_ISET / R_ISET
      = 300 A·Ω / 13.68 kΩ
      ≈ 21.9 mA
```

The board's default ISET setting must be disconnected so that the
effective ISET resistance is 13.68 kΩ. For settings below 50 mA,
the BQ25185 datasheet also recommends an additional ISET RC network
using a 50 pF capacitor. Both the circuit modification and the actual
charging current remain to be verified.

Before connecting the cell for charging, verify its manufacturer's permitted
charge current, connector polarity, protection arrangement, and the actual
charger settings. Battery capacity alone does not establish charging safety.

References: [Adafruit BQ25185 guide](https://learn.adafruit.com/adafruit-bq25185-usb-dc-solar-lithium-ion-polymer-charger/pinouts)
and [TI BQ25185 datasheet](https://www.ti.com/lit/ds/symlink/bq25185.pdf).

## 5. Position Validity

A fresh GNSS fix is a measured position. When GNSS is unavailable but a
Sigfox message reaches the network, approximate Sigfox network location
may be considered together with the last valid fix. The location source,
fix age, and estimated uncertainty must remain distinguishable.

Without a received Sigfox message, the server has no new position information.
Network-derived locations can have large uncertainty, especially in areas
with sparse or unfavorable base-station reception.

Reference: [KCCS Atlas Native](https://www.kccs.co.jp/sigfox/service/atlas/).

## 6. Verification

The 15-minute cycle will be characterized using the following measurements:

| Metric | Result |
| --- | --- |
| Cycle-averaged current (mA) | Not measured |
| Cycle-averaged power (mW) | Not measured |
| Charge per 15-minute cycle (mAh) | Not measured |
| GNSS acquisition time and success rate | Not measured |
| Sigfox server receipt rate | Not measured |
| Seven-day battery runtime | Not verified |

Each result will include the hardware and firmware revision, power source,
instruments, measurement procedure, and environmental conditions.
