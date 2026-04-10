# InkTime v5

A low-power e-paper smartwatch based on the Nordic nRF52840 SoC, featuring Bluetooth LE, haptic feedback, and a 3-button interface.

## Block Diagram

```
                        ┌─────────────────────────────────────────┐
                        │              nRF52840                    │
                        │                                          │
 USB-C ──► BQ25180 ──►  │  SPI ──► E-Paper Display (2.9")         │
           (LiPo        │  I2C ──► BMA423 (IMU)                   │
           Charger)      │  I2C ──► DRV2605 (Haptic)               │
              │          │  I2C ──► MAX17048 (Fuel Gauge)          │
              ▼          │  I2C ──► BQ25180 (Charger)              │
           RT6160 ──────►│  I2C ──► RT6160 (DC/DC)                 │
           (DC/DC        │  GPIO ► SW_UP, SW_ENT, SW_DN (Buttons)  │
           3.3V)         │  SWD ──► TC2030 Debug Connector         │
                        │  ANT ──► 2450AT18B100E (BLE Antenna)    │
                        └─────────────────────────────────────────┘
```

## Bill of Materials

| Reference | Component | Value | Description | JLC Part | Datasheet |
|-----------|-----------|-------|-------------|----------|-----------|
| U1 | nRF52840 | - | BLE SoC, main MCU | - | - |
| U2 | RT6160AWSC | - | DC/DC Buck Converter 3.3V | - | - |
| U3 | MAX17048G+T10 | - | LiPo Fuel Gauge | - | - |
| U4 | BQ25180YBGR | - | LiPo Charger | - | - |
| U5 | DRV2605YZFR | - | Haptic Driver | - | - |
| U6 | BMA423 | - | IMU (Accelerometer) | - | - |
| U7 | SI1308EDL-T1-GE3 | - | P-Channel MOSFET (EPD Power Switch) | - | - |
| D1 | MBR0530 | - | Schottky Diode (DC/DC) | - | -.pdf) |
| D2, D3 | MBR0530 | - | Schottky Diode (EPD Drive) | - | -.pdf) |
| D4 | USBLC6-2SC6Y | - | USB ESD Protection | - | - |
| L1 | 2450AT18B100E | - | 2.4GHz BLE Chip Antenna | - | - |
| L2 | FTC252012SR47MBCA | 4.7µH | Inductor (DC/DC) | - | - |
| J1 | KH-TYPE-C-16P | - | USB-C Connector | - | - |
| J2 | TC2030-IDC | - | SWD Debug Connector | - | - |
| J3 | 503480-2400 | - | E-Paper FPC Connector 24-pin | - | - |
| SW1 | SW_UP | - | Tactile Button | - | - |
| SW2 | SW_ENT | - | Tactile Button | - | - |
| SW3 | SW_DN | - | Tactile Button | - | - |
| BAT1 | AKYGA LP502030 | 3.7V/250mAh | LiPo Battery | - | - |

## Hardware Description

### Power Architecture

The device is powered by a **3.7V LiPo battery (AKYGA LP502030, 250mAh)**. Charging is handled by the **BQ25180** via USB-C (5V VBUS). The BQ25180 communicates with the nRF52840 over **I²C** (PMC_INT alert pin) for charge status monitoring.

The **RT6160 DC/DC buck converter** steps down VBAT to **3.3V** for the nRF52840 and all peripherals. It is also controlled via I²C from the nRF (SW1/SW2 pins for dynamic voltage scaling).

Battery state of charge is monitored by the **MAX17048 fuel gauge** over I²C (ALERT pin connected to nRF GPIO).

The **E-Paper display** has a separate power rail (**EPD_3V3**) switched by a P-channel MOSFET (SI1308EDL) controlled by a GPIO on the nRF52840. This allows the display to be fully powered off between updates to save energy.

### nRF52840 Pin Assignment

| nRF52840 Pin | Net | Peripheral | Interface |
|-------------|-----|-----------|-----------|
| P0.26 / XL1 | EPD_CLK | E-Paper Display | SPI CLK |
| P0.27 / XL2 | EPD_MOSI | E-Paper Display | SPI MOSI |
| P0.01 | EPD_CS | E-Paper Display | SPI CS |
| P0.00 | EPD_DC | E-Paper Display | SPI DC |
| P0.09 | EPD_RST | E-Paper Display | GPIO |
| P0.10 | EPD_BUSY | E-Paper Display | GPIO Input |
| P0.26 | SCL | I²C Bus (BQ25180, RT6160, MAX17048, DRV2605) | I²C SCL |
| P0.25 | SDA | I²C Bus | I²C SDA |
| P0.06 | IMU_INT1 | BMA423 | GPIO Input |
| P0.05 | IMU_INT2 | BMA423 | GPIO Input |
| P0.07 | HAPTIC_EN | DRV2605 | GPIO Enable |
| P0.08 | EPD_PWR_EN | SI1308EDL Gate | GPIO |
| P0.11 | SW_UP | Button Up | GPIO Input |
| P0.12 | SW_ENT | Button Enter | GPIO Input |
| P0.13 | SW_DN | Button Down | GPIO Input |
| P0.18 | RESET | Reset | Active Low |
| SWDIO | SWDIO | TC2030 SWD | Debug |
| SWDCLK | SWDCLK | TC2030 SWD | Debug |
| ANT | ANT | 2450AT18B100E | RF 2.4GHz |

### E-Paper Display

The E-Paper display connects via a **24-pin FPC connector (503480-2400)**. The drive circuit uses a boost converter based on the SI1308 MOSFET and MBR0530 diodes to generate the high voltages required for the EPD gate drivers (PREVGH ~15V, PREVGL ~-15V). Communication is via SPI.

### IMU

The **BMA423** accelerometer connects via I²C and provides step counting, activity detection, and wrist-tilt detection. Two interrupt lines (IMU_INT1, IMU_INT2) allow wake-on-motion functionality to minimize power consumption.

### Haptic Feedback

The **DRV2605** haptic driver connects via I²C and drives an ERM or LRA vibration motor. It is enabled via a dedicated GPIO from the nRF52840 (HAPTIC_EN).

### BLE Antenna

The **2450AT18B100E** chip antenna is placed at the corner of the PCB with a full copper keepout zone on all layers beneath it, as required by the antenna datasheet for optimal RF performance.

### USB & ESD Protection

The **KH-TYPE-C-16P** USB-C connector includes CC1/CC2 pins for USB PD negotiation. The **USBLC6-2SC6Y** TVS diode array provides ESD protection on the D+/D- lines. A 1kΩ resistor limits inrush current on VBUS.

### SWD Debug Interface

Programming and debugging is done via a **TC2030-IDC Tag-Connect** footprint, exposing SWDIO, SWDCLK, RESET, 3V3, and GND. No physical connector is required — the Tag-Connect pogo-pin cable clips directly onto the PCB pads.

## Power Consumption Estimates

| Mode | Current | Notes |
|------|---------|-------|
| BLE Advertising | ~0.5mA avg | 1s interval |
| BLE Connected | ~1.5mA avg | Active connection |
| Deep Sleep | ~3µA | nRF System OFF, RTC wake |
| E-Paper Refresh | ~20mA peak | During display update only |
| Haptic Active | ~50mA peak | Motor running |
| **Estimated avg (watch mode)** | **~0.3mA** | 1 EPD refresh/min, BLE adv |

Estimated battery life with 250mAh: **~30 days** in watch mode.

## PCB

- **Layers:** 3 (Top signal, GND plane, Bottom signal)
- **Dimensions:** ~54 × 40mm
- **Fabrication:** JLCPCB 4-layer standard
- **Min trace width:** 0.15mm
- **Min clearance:** 0.15mm

## License

Apache License 2.0 — see [LICENSE](LICENSE)
