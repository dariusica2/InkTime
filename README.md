# InkTime Smartwatch – Open Source E-Paper Watch

InkTime is an open-source, low-cost smartwatch built around the nRF52840.  
This repository contains the **Hardware design** (schematic, PCB, 3D models, BOM, and manufacturing files).

## Hardware Functionality

### 1. Power Management
- **USB‑C (J4)** provides VBUS (5V) to the **BQ25180** linear charger.
- **BQ25180** charges a single‑cell Li‑Po battery (connected via test pads `TP_VBAT` / `TP_BAT_GND`).  
  It outputs **SYS** (≈4.4V) which feeds the **RT6160** buck‑boost.
- **RT6160AWSC** (I²C programmable) generates a regulated **3.3V** rail for the whole system.  
  Output set via `VSEL` resistor divider (2.2k / 0.47Ω – jumper configurable).
- **MAX17048** fuel gauge monitors battery voltage and reports via I²C (`ALERT` pin interrupts the MCU).

### 2. Main Microcontroller – nRF52840
- 32‑bit ARM Cortex‑M4F, 1MB Flash, 256kB RAM.
- Interfaces used:
  - **USB 2.0** (D+/D‑) via ESD protection (USBLC6‑2).
  - **I²C** (SCL/SDA) for charger, fuel gauge, IMU, haptic driver, and buck‑boost.
  - **SPI** to the e‑paper display (SCK, MOSI, CS, DC, RST, BUSY).
  - **SWD** for programming/debugging (TC2030 connector).
  - **32MHz** crystal for RF and **32.768kHz** crystal for RTC.
  - **GPIOs** for buttons, haptic enable, and interrupt inputs.

### 3. E‑Paper Display (EPD) Interface
- Connected via 24‑pin FPC connector **J1** (Molex 503480‑2400).
- SPI signals: `SCK`, `MOSI` (from nRF52), `CS`, `DC`, `RST`, `BUSY`.
- Power for the e‑paper panel is provided by the **EPD_3V3** rail, switched by a P‑ch MOSFET (`Q1`) driven by `P1.01` (active low).  
  A 10k pull‑up (`R_PWR_EPD`) keeps the display off until the MCU pulls the gate low.
- Decoupling capacitors (1µF) on each power pin of the FPC.

### 4. Sensors & Actuators
- **BMA423** accelerometer (I²C, address configurable via `SDO`).  
  Interrupts: `INT1` → `P0.08`, `INT2` → `P1.08`.  
  VDDIO = 3.3V, VDD = 3.3V.
- **DRV2605** haptic driver (I²C, address 0x5A).  
  Drives a linear resonant actuator (LRA) or eccentric rotating mass (ERM) via `OUT+` / `OUT-`.  
  Enable pin (`EN`) controlled by `P0.12`.  
  Decoupling: 1µF on VDD, 4.7µF on REG.

### 5. User Input
- Three tactile buttons (Up, Enter, Down) connected to `P0.13`, `P0.14`, `P1.02` with internal pull‑ups (external 10k resistors also present).  
  Pins are pulled to GND when pressed.

### 6. Debug & Programming
- **TC2030‑IDC** (Tag‑Connect) footprint provides SWD: `SWDCLK`, `SWDIO`, `SWO`, `RESET`, 3.3V, GND.
- Test points expose all critical signals: `SWDCLK`, `SWDIO`, `SWO`, `RESET`, `SCL`, `SDA`, `3.3V`, `VBAT`, `GND`, `VREG`.

### 7. RF Section
- **2450AT18B100E** chip antenna matched with a **π‑network** (C3, C22, L1, C4).  
- Antenna placed on the board edge, with **PCB cutout underneath** (no ground or traces under antenna area).
- 50Ω impedance matching ensured by discrete L/C values (3.9nH, 1pF, 1pF, 1pF).

### 8. USB Interface
- **USB‑C** connector (KH‑TYPE‑C‑16P) routed to nRF52840 D+/D‑ via **USBLC6‑2** ESD protection.
- CC lines pulled down with 5.1k resistors for source‑detection (host mode).
- Shield connected to GND.

## Compliance & Design Rules

The PCB was designed with the following constraints (verified by DRC):

| Rule | Notes |
|------|-------|
| **Components on TOP only** | No components on bottom side |
| **4‑layer board** | Top + Power + Bottom routing with Ground pour on GND layer |
| **Power trace width** | 0.3mm |
| **Signal trace width** | 0.15mm |
| **No right‑angle traces** | 45° or curved corners only |
| **Decoupling caps near ICs** | 100nF within 2mm of each power pin |
| **Antenna keep‑out** | No ground or traces under antenna (cutout on PCB) |
| **Via stitching** | Multiple vias between TOP and BOTTOM ground planes, especially near RF |
| **ERC** | Only "Only INPUT pins on NET ID" warnings ignored (by design) |
| **DRC** | Using JLCPCB 2‑layer rules (clearance 5mil, min drill 0.2mm) |
| **Silkscreen** | Component names only (no values), readable, test points labeled |
| **Mechanical fit** | PCB fits into provided enclosure, USB and buttons align |

## Design Decisions & Known Issues

### Accepted DRC/ERC warnings
- **“Only INPUT pins on NET ID”** – ignored as per project guidelines.
- **Dimension errors** caused by USB‑C connector and buttons extending slightly beyond board outline are **ignored** (they fit inside the given enclosure).
- **Silkscreen overlaps** on small 0201 components – unavoidable due to density; names still legible.

### Routing choices
- **Power lines** traces are 0.3mm to handle charging current.
- **Via uses** on 4‑layer board – only used for transitioning between layers when unavoidable.

### Battery connection
- The Li‑Po cell is soldered directly to two test pads (`TP_VBAT`, `TP_BAT_GND`) – no JST connector, saving space.

### Review feedback (DVT stage)
- **Issue resolved**: PCB under antenna not cropped - corpped PCB under antenna
- **Issue resolved**: right angles in routing - using 45 degree angles
- **Issue resolved**: antenna keep‑out area not fully cleared - added via ground stitching around the antenna path
