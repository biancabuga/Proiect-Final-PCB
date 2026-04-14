# InkTime Smartwatch — Hardware Documentation

> Open-source smartwatch with e-Paper display, Nordic nRF52840 SoC, and BLE 5.0 connectivity.

---

## Table of Contents

1. [Block Diagram](#block-diagram)
2. [Bill of Materials](#bill-of-materials)
3. [Hardware Description](#hardware-description)
4. [Pin Map — nRF52840](#pin-map--nrf52840)
5. [Power Budget](#power-budget)
6. [Debug & Test Points](#debug--test-points)
7. [Design Notes & Decisions](#design-notes--decisions)

---

## Block Diagram

```
                          ┌─────────────────────────────────────────────┐
                          │              nRF52840 (U1)                  │
                          │   ARM Cortex-M4F @ 64 MHz / BLE 5.0        │
                          │                                             │
   ┌──────────┐  SPI      │  P0.02 SCK ──────────────────────────────► │──► e-Paper Display (J1)
   │ e-Paper  │◄──────────│  P0.03 MOSI                                │    503480-2400 FPC 24-pin
   │ Display  │           │  P0.05 EPD_CS                              │
   └──────────┘           │  P0.15 EPD_DC                              │
                          │  P0.16 EPD_RST                             │
   ┌──────────┐           │  P0.17 EPD_BUSY                            │
   │ BQ25180  │◄── I²C ───│  P0.06 SDA / P0.07 SCL                    │◄── USB-C (J4)
   │  (IC1)   │  0x6A     │  P0.11 PMIC_INT ◄────────────────────     │    KH-TYPE-C-16P
   └──────────┘           │                                             │
                          │                                             │
   ┌──────────┐           │                      ┌─────────────────────┤
   │ MAX17048 │◄── I²C ───│  0x36                │   Power Rails       │
   │  (U3)    │           │  P0.10 ALERT ◄───    │   VBUS (5V) USB     │
   └──────────┘           │                      │   VBAT (3.0–4.2V)   │
                          │                      │   VREG (~3.6V)      │
   ┌──────────┐           │                      │   3V3 (3.3V)        │
   │  BMA423  │◄── I²C ───│  0x18                │   EPD_3V3 (SW)      │
   │  (IC3)   │           │  P0.08 INT1 ◄───     └─────────────────────┤
   └──────────┘           │  P1.08 INT2 ◄───                           │
                          │                                             │
   ┌──────────┐           │                                             │
   │ DRV2605  │◄── I²C ───│  0x5A                                      │
   │  (IC2)   │──► LRA    │  P0.12 EN                                  │
   └──────────┘           │                                             │
                          │                                             │
   ┌──────────┐           │  P0.13 SW_UP                               │
   │ Buttons  │──── GPIO ─│  P0.14 SW_ENT                              │
   │ SW_UP    │           │  P1.02 SW_DN                               │
   │ SW_ENT   │           │                                             │
   │ SW_DN    │           │  SWDIO / SWDCLK ──────────────────────────►│──► SWD Connector (J2)
   └──────────┘           │  P1.00 SWO                                 │    TC2030-IDC
                          │                                             │
   ┌──────────┐           │  ANT (H23) ──────────────────────────────► │──► 2450AT18B100E Antenna
   │RT6160AWSC│◄─ VREG ───│  3V3 ──────────────────────────────────►  │    π matching network
   │  (IC9)   │──► 3V3    │                                             │
   └──────────┘           │  X1: 32 MHz HF crystal                     │
                          │  X2: 32.768 kHz LF/RTC crystal             │
                          └─────────────────────────────────────────────┘
```

---

## Bill of Materials

| Ref | Component | Description | Package | Quantity | JLC Parts | Datasheet |
|-----|-----------|-------------|---------|----------|-----------|-----------|
| U1 | nRF52840-QFAA | SoC BLE 5.0 + ARM Cortex-M4F | AQFN-73 | 1 | [C190794](https://jlcpcb.com/partdetail/NordicSemiconductor-nRF52840QFAA/C190794) | [Datasheet](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.7.pdf) |
| IC1 | BQ25180YBGR | Li-Po charger, I²C, 350 mA | DSBGA-9 | 1 | [C2681576](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C2681576) | [Datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| IC2 | DRV2605YZFR | Haptic driver LRA/ERM, I²C | DSBGA-12 | 1 | [C116408](https://jlcpcb.com/partdetail/TexasInstruments-DRV2605YZFR/C116408) | [Datasheet](https://www.ti.com/lit/ds/symlink/drv2605.pdf) |
| IC3 | BMA423 | 3-axis IMU + pedometer, I²C | LGA-12 | 1 | [C2681895](https://jlcpcb.com/partdetail/Bosch-BMA423/C2681895) | [Datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma423-ds000.pdf) |
| IC9 | RT6160AWSC | DC-DC buck 3.3V | WLCSP-16 | 1 | [C2836542](https://jlcpcb.com/partdetail/RichTek-RT6160AWSC/C2836542) | [Datasheet](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-00.pdf) |
| U3 | MAX17048G+T10 | LiPo fuel gauge, I²C | SOT-23-6 | 1 | [C82490](https://jlcpcb.com/partdetail/MaximIntegrated-MAX17048GT10/C82490) | [Datasheet](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf) |
| Q1 | DMG2305UX-7 | PMOS 20V/4A (EPD power switch) | SOT-323 | 1 | [C145103](https://jlcpcb.com/partdetail/Diodes-DMG2305UX7/C145103) | [Datasheet](https://www.diodes.com/assets/Datasheets/DMG2305UX.pdf) |
| Q3 | SI1308EDL-T1-GE3 | NMOS 20V/1.3A (GDR switch) | SC-70-3 | 1 | [C145890](https://jlcpcb.com/partdetail/VishayIntertech-SI1308EDLT1GE3/C145890) | [Datasheet](https://www.vishay.com/docs/66966/si1308edl.pdf) |
| D3 | USBLC6-2SC6Y | USB ESD protection | SOT-363 | 1 | [C2687116](https://jlcpcb.com/partdetail/STMicroelectronics-USBLC62SC6Y/C2687116) | [Datasheet](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| D4, D5 | MBR0530 | Schottky diode 30V/0.5A | SOD-123 | 2 | [C82513](https://jlcpcb.com/partdetail/OnSemiconductor-MBR0530T1G/C82513) | [Datasheet](https://www.onsemi.com/pdf/datasheet/mbr0530t1-d.pdf) |
| ANT1 | 2450AT18B100E | Ceramic antenna 2.45 GHz | 0603 | 1 | [C5376480](https://jlcpcb.com/partdetail/JohansonTechnology-2450AT18B100E/C5376480) | [Datasheet](https://www.johansontechnology.com/datasheets/2450AT18B100E/2450AT18B100E.pdf) |
| J1 | 503480-2400 | FPC connector 24-pin 0.5mm | SMD | 1 | [C2938877](https://jlcpcb.com/partdetail/Molex-5034802400/C2938877) | [Datasheet](https://www.molex.com/en-us/products/part-detail/5034802400) |
| J4 | KH-TYPE-C-16P | USB-C connector 16-pin | SMD | 1 | [C2765186](https://jlcpcb.com/partdetail/KOREAN_HROPARTS_ELEC-KHTYPEC16P/C2765186) | [Datasheet](https://datasheet.lcsc.com/lcsc/2206111530_Korean-Hroparts-Elec-TYPE-C-31-M-12_C165948.pdf) |
| J2 | TC2030-IDC | Tag-Connect SWD 6-pin | THT (no pins) | 1 | — | [Datasheet](https://www.tag-connect.com/wp-content/uploads/bsk-pdf-manager/TC2030-IDC-NL_Datasheet_8.pdf) |
| X1 | — | Crystal 32 MHz | SMD 3225 | 1 | [C9002](https://jlcpcb.com/partdetail/C9002) | — |
| X2 | — | Crystal 32.768 kHz | SMD 3215 | 1 | [C32346](https://jlcpcb.com/partdetail/C32346) | — |
| L1 | — | Inductor 3.9 nH (ant. match) | 0402 | 1 | — | — |
| L2 | — | Inductor 10 µH (RF decoupling) | 0402 | 1 | — | — |
| L3 | — | Inductor 15 nH | 0402 | 1 | — | — |
| L5 | — | Inductor 68 µH (EPD pump) | 0402 | 1 | — | — |
| L7 | FTC252012SR47MBCA | Inductor 0.47 µH (DC-DC) | 1008 | 1 | [C408499](https://jlcpcb.com/partdetail/TDK-FTC252012SR47MBCA/C408499) | — |
| R1_USB, R2_USB | — | Resistor 5.1 kΩ (CC pull-down) | 0201 | 2 | — | — |
| R17, R18 | — | Resistor 3.3 kΩ (I²C pull-up) | 0201 | 2 | — | — |
| R5, R7, R8 | — | Resistor 10 kΩ (button pull-up) | 0201 | 3 | — | — |
| R3, R2, R4 | — | Resistor 0 Ω (IMU config) | 0201 | 3 | — | — |
| C1, C2 | — | Capacitor 12 pF (X1 load) | 0201 | 2 | — | — |
| C17, C18 | — | Capacitor 12 pF (X2 load) | 0201 | 2 | — | — |
| C3, C4 | — | Capacitor 1 pF (ant. match) | 0201 | 2 | — | — |
| C7, C8, C12 | — | Capacitor 100 nF (decoupling) | 0201 | 3+ | — | — |
| C14 | — | Capacitor 4.7 µF (VDD decoupling) | 0402 | 1 | — | — |
| C25, C33 | — | Capacitor 22 µF (DC-DC output) | 0402 | 2 | — | — |
| C30, C31, C29 | — | Capacitor 1 µF (button debounce) | 0201 | 3 | — | — |
| C2-EP-DR | — | Capacitor 4.7 µF / 25V (EPD pump) | 0402 | 1 | — | — |
| C1-EP-DR | — | Capacitor 10 µF (EPD pump) | 0402 | 1 | — | — |

> All resistors are SMD 0201. Capacitors ≤ 100 nF are 0201; capacitors > 100 nF are 0402, unless otherwise specified in the schematic.

---

## Hardware Description

### Microcontroller — nRF52840 (U1)

The nRF52840 is the central SoC of the InkTime watch. It integrates an ARM Cortex-M4F core running at up to 64 MHz, 1 MB of Flash, 256 KB of RAM, a native BLE 5.0 radio, and a USB Full-Speed 2.0 interface — all in a 7×7 mm AQFN-73 package.

Supply voltage is 3.3V applied to VDD pins (B1, A22, W1, AD14, AD23) and VDDH (Y2). Local decoupling consists of C14 (4.7 µF), multiple 100 nF capacitors, and C25/C33 (22 µF each) on the DC-DC output.

The nRF52840 orchestrates all peripherals: it drives the e-Paper display via SPI, reads sensor and power data via a shared I²C bus, manages USB enumeration directly, and controls the EPD power switch, haptic driver enable, and button inputs via GPIO.

### Power Architecture

The board has five distinct power domains:

| Rail | Source | Voltage | Consumers |
|------|--------|---------|-----------|
| `VBUS` | USB-C J4 | 5 V | BQ25180 IN, ESD D3, U1 VBUS detection |
| `VBAT` | Li-Po battery | 3.0–4.2 V | BQ25180 BAT, MAX17048 VDD/CELL |
| `VREG` | BQ25180 SYS | ~3.6 V | RT6160 VIN/EN |
| `3V3` | RT6160 VOUT | 3.3 V | U1 VDD, all ICs, Q1 Source |
| `EPD_3V3` | Q1 PMOS | 3.3 V (switched) | J1 e-Paper display |

**BQ25180YBGR (IC1)** is the Li-Po charger. It communicates via I²C at address 0x6A (connected to P0.06/P0.07 with 3.3 kΩ pull-ups) and provides a maximum charge current of 350 mA. The `!INT` signal is routed to P0.11 for event notification (charge complete, fault).

**RT6160AWSC (IC9)** is a DC-DC buck converter that generates the main 3.3V rail from VREG. The switching inductor is L7 (0.47 µH), and the output filter uses two 22 µF capacitors (C25 + C33).

**MAX17048G+T10 (U3)** is the Li-Po fuel gauge. It uses the ModelGauge algorithm to estimate state-of-charge (SOC) and communicates on the shared I²C bus at address 0x36. Its `!ALERT` pin is connected to P0.10 (NFC2) for low-battery alerts.

**EPD power switch:** The e-Paper display is powered through a PMOS transistor Q1 (DMG2305UX), controlled by P1.01 (LOW = ON). An NMOS Q3 (SI1308EDL) handles the GDR line (gate drive return) of the display. This allows the MCU to fully cut power to the display when not in use, reducing standby consumption.

### e-Paper Display (J1)

The display connects through a 24-pin FPC connector (503480-2400) and uses a write-only SPI interface plus several GPIO control lines. SPI clock and data are P0.02 and P0.03 respectively. An internal charge pump circuit generates the higher voltages needed for EPD drive, using L5 (68 µH), C1-EP-DR, C2-EP-DR, and two Schottky diodes D4/D5 (MBR0530).

The display's power rail (EPD_3V3) is switched via Q1, controlled by P1.01. This is essential for e-Paper which otherwise leaks small currents continuously. The bistable nature of the display means refresh power (~25 mA × 500 ms ≈ 3.5 µAh per update) is negligible over the battery's lifetime.

### Inertial Measurement Unit — BMA423 (IC3)

The BMA423 is a compact 3-axis accelerometer with on-chip step counting and wrist-tilt gesture detection — making it ideal for a smartwatch. It communicates via I²C at address 0x18 (CSB tied to 3V3, SDO to GND via 0 Ω R3). Two interrupt lines (INT1 on P0.08, INT2 on P1.08) allow asynchronous notification of gesture or step events without polling. Typical power: 1.5 µA in standby, 130 µA active.

### Haptic Driver — DRV2605 (IC2)

The DRV2605 drives the Panasonic EVP-AKE31A LRA motor and contains 123 pre-programmed haptic effects in ROM. It operates in I²C-only mode (IN/TRIG tied to GND) at address 0x5A. The EN pin (P0.12) allows the MCU to fully power-gate the driver between haptic events. Motor terminals are exposed as test points TP_OP and TP_ON.

### USB-C Interface and ESD Protection (J4, D3)

The KH-TYPE-C-16P connector is configured as a power Sink only. Resistors R1_USB and R2_USB (5.1 kΩ each) pull down the CC1/CC2 lines to GND, signaling 500 mA Sink capability per the USB-C specification. ESD protection on D+ and D− is provided by the USBLC6-2SC6Y (D3), a dedicated USB ESD suppressor rated to IEC61000-4-2 Level 4. Data lines route directly to the nRF52840's USB PHY pads (AD4, AD6).

### BLE Antenna and RF Section

The Johanson 2450AT18B100E is a 50 Ω ceramic chip antenna operating at 2.45 GHz. A π-network matching circuit (L1 = 3.9 nH series, C3 = C4 = 1 pF shunt) adapts the nRF52840's internal RF output impedance to the antenna. L2 (10 µH) provides DC isolation on the DCC line to decouple the internal radio DC-DC converter from the rest of the board.

Per PCB layout rules, the antenna must be placed at the board edge and the PCB must be cut out underneath it — no copper fills, no signal routing under the antenna.

### Crystals

Two oscillators support the nRF52840's clock domains:

- **X1 — 32 MHz**: High-frequency crystal for the main CPU and radio. Connected to XC1 (B24) and XC2 (A23), with 12 pF load capacitors C1 and C2.
- **X2 — 32.768 kHz**: Low-frequency crystal for the RTC. Connected to XL1/P0.00 (D2) and XL2/P0.01 (F2), with 12 pF load capacitors C17 and C18.

### Buttons

Three user buttons (SW_UP, SW_ENT, SW_DN) use an active-low scheme: each has a 10 kΩ pull-up resistor to 3V3 and a 1 µF debounce capacitor to GND.

| Button | GPIO | Pull-up | Debounce cap |
|--------|------|---------|--------------|
| SW_UP | P0.13 | R5 10 kΩ | C30 1 µF |
| SW_ENT | P0.14 | R8 10 kΩ | C31 1 µF |
| SW_DN | P1.02 | R7 10 kΩ | C29 1 µF |

### Shared I²C Bus

All four I²C peripherals share a single bus on P0.06 (SDA) and P0.07 (SCL), pulled up to 3V3 via R17 = R18 = 3.3 kΩ. The bus operates at 400 kHz (Fast Mode). Addresses are well-separated so there are no conflicts.

| Device | Ref | Address |
|--------|-----|---------|
| BQ25180 charger | IC1 | 0x6A |
| DRV2605 haptic | IC2 | 0x5A |
| BMA423 IMU | IC3 | 0x18 |
| MAX17048 fuel gauge | U3 | 0x36 |

---

## Pin Map — nRF52840

| GPIO | Package Pad | Connected To | Function |
|------|-------------|--------------|----------|
| P0.00 / XL1 | D2 | X2 pin 1 | 32.768 kHz crystal |
| P0.01 / XL2 | F2 | X2 pin 2 | 32.768 kHz crystal |
| P0.02 / AIN0 | A12 | J1 pin 13 | SPI SCK → e-Paper |
| P0.03 / AIN1 | B13 | J1 pin 14 | SPI MOSI → e-Paper |
| P0.05 / AIN3 | K2 | J1 pin 12 | EPD_CS (active LOW) |
| P0.06 | L1 | IC1/2/3, U3 | I²C SDA (shared) |
| P0.07 | M2 | IC1/2/3, U3 | I²C SCL (shared) |
| P0.08 | N1 | IC3 INT1 | IMU interrupt 1 (tilt/step) |
| P0.10 / NFC2 | J24 | U3 !ALERT | Fuel gauge low-battery alert |
| P0.11 | T2 | IC1 !INT | PMIC event interrupt |
| P0.12 | U1 | IC2 EN | Haptic driver enable |
| P0.13 | AD8 | SW_UP | Button UP (active LOW) |
| P0.14 | AC9 | SW_ENT | Button ENTER (active LOW) |
| P0.15 | AD10 | J1 pin 11 | EPD_DC (Data/Command) |
| P0.16 | AC11 | J1 pin 10 | EPD_RST (active LOW) |
| P0.17 | AD12 | J1 pin 9 | EPD_BUSY (input) |
| P0.18 / RESET | AC13 | J2 pin 6 | Chip reset |
| P1.00 | AD22 | TP_SWO | SWD trace output |
| P1.01 | Y23 | Q1 Gate | EPD power switch (LOW = ON) |
| P1.02 | W24 | SW_DN | Button DOWN (active LOW) |
| P1.08 | P2 | IC3 INT2 | IMU interrupt 2 |
| SWDIO | AC24 | J2 pin 2 | SWD data |
| SWDCLK | AA24 | J2 pin 4 | SWD clock |
| D+ | AD6 | D3 → J4 | USB FS D+ |
| D− | AD4 | D3 → J4 | USB FS D− |
| VBUS | AD2 | J4, IC1 IN | USB VBUS detection |
| ANT | H23 | L1, C3, C4 | BLE antenna 50 Ω |
| XC1 | B24 | X1 pin 1 | 32 MHz HF crystal |
| XC2 | A23 | X1 pin 3 | 32 MHz HF crystal |

Pins not listed (P0.04, P0.09, P0.19–P0.31, P1.03–P1.07, P1.09–P1.15) are left unconnected and reserved.

---

## Power Budget

Scenario: static e-Paper display, BLE advertising at 1 Hz, RTC active, IMU in pedometer mode.

| Component | Estimated Current |
|-----------|-----------------|
| nRF52840 (sleep + RTC + BLE adv @ 1 Hz) | ~200 µA |
| BMA423 (low-power pedometer) | ~130 µA |
| MAX17048 (active continuously) | ~23 µA |
| RT6160 quiescent | ~30 µA |
| BQ25180 standby (no USB) | ~2 µA |
| DRV2605 standby | ~3 µA |
| **Total (no display refresh)** | **~388 µA** |

With a **100 mAh Li-Po** battery → estimated autonomy **≈ 10 days**.

e-Paper refresh consumes approximately 25 mA for 500 ms ≈ 3.5 µAh per update. Even with 100 refreshes per day, this adds only 350 µAh/day, which is negligible compared to the ~9.3 mAh/day background consumption.

---

## Debug & Test Points

**SWD Connector:** TC2030-IDC (J2, Tag-Connect footprint without permanent pins). Signals: SWDIO (AC24), SWDCLK (AA24), RESET (AC13), SWO (P1.00/AD22), 3.3V, GND.

All test points are labeled in silkscreen with the signal name. Available TPs:

| Test Point | Signal |
|------------|--------|
| TP_3V3 | 3.3V regulated |
| TP_VBAT | Battery voltage |
| TP_VREG | BQ25180 SYS output |
| TP_GND | Ground |
| TP_SCL | I²C clock |
| TP_SDA | I²C data |
| TP_SWDIO | SWD data |
| TP_SWDCLK | SWD clock |
| TP_SWO | Trace output |
| TP_RESET | MCU reset |
| TP_OP | Haptic motor OUT+ |
| TP_ON | Haptic motor OUT− |
| TP_BAT_GND | Battery negative terminal |

Battery connects directly to TP_VBAT and TP_BAT_GND (no JST connector used, to save board space per project specification).

---

## Design Notes & Decisions

- **I²C pull-up value (3.3 kΩ):** With four devices on the bus and a target of 400 kHz, a 3.3 kΩ pull-up gives acceptable rise times while not overloading the 3V3 rail.

- **EPD power switch (PMOS Q1):** Rather than leaving the display permanently powered, the PMOS switch allows the MCU to cut EPD_3V3 between refreshes. The bistable nature of e-Paper means image retention with zero power, so keeping the display rail live is wasteful. LOW on P1.01 turns Q1 on (PMOS convention).

- **NFC pins re-used as GPIO (P0.10):** The nRF52840 allows its NFC antenna pins to be configured as GPIO when NFC is not needed. P0.10 is used for U3 !ALERT. This is acceptable since InkTime does not require NFC functionality.

- **ERC error "Only INPUT pins on NET ID":** This known KiCad ERC warning is ignored per project guidelines, as it results from how net IDs are handled in the schematic tool and does not represent a real circuit issue.

- **CC resistors 5.1 kΩ to GND:** Per the USB-C specification, a Sink device must pull both CC lines to GND through 5.1 kΩ resistors to advertise itself as a powered device requesting up to 500 mA. No active PD controller is required for basic 5V charging.

- **PCB thickness 1 mm:** The enclosure constraint specifies 1 mm PCB thickness (not the standard 1.6 mm). This affects via drill sizing and trace impedance slightly but is well within manufacturer capabilities (e.g., JLCPCB supports 1 mm).

- **Antenna placement:** The Johanson 2450AT18B100E must sit at the board perimeter with a PCB cutout underneath. No copper (neither planes nor traces) is permitted in this area. Via stitching is applied around the RF section to improve ground isolation.

- **Component placement:** All components are placed on the TOP layer only. Bottom layer is used exclusively for routing. A ground plane is present on both TOP and BOTTOM layers; via stitching connects them, especially around the RF section.

---

## License

This project is licensed under the [Apache License 2.0](LICENSE).
