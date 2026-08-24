# AirGradient Open Air Max — Hardware

> KiCad hardware design files for the [AirGradient Open Air Max](https://www.airgradient.com/professional/products/open-air-max/), a professional outdoor air quality monitor.

## Overview

Open Air Max is AirGradient's professional outdoor air quality monitor, designed for autonomous, off-grid operation. It measures PM2.5 (2× Plantower PMS5003), CO₂ (SenseAir Sunlight NDIR, 0–5,000 ppm), TVOCs/NOₓ (Sensirion SGP41), NO₂ (AlphaSense A43F) and O₃ (AlphaSense A431), plus temperature and humidity (Sensirion SHT40), in a UV-resistant, weatherproof ASA enclosure with wall or pole mounting (model O-M-1PPSTON-CE; CE, RoHS, REACH, and FCC certified).

Data is reported over Wi-Fi (2.4 GHz IEEE 802.11 b/g/n) or 4G cellular (LTE Cat 1). Power comes from a 10 W solar panel charging a user-replaceable pack of three 3,500 mAh Li-ion 18650 cells in series (3S), managed by a dedicated battery management system and a low-power design throughout.

This repository contains the monitor's main PCB: the ESP32-C6 MCU, solar/USB-C charging and power path, battery management, and the interfaces the sensor modules and the cellular CE card plug into — complete with schematics, PCB layout, bundled 3D models, and the production output set.

## Design structure

The design is a single 4-layer board (signal / power plane / ground plane / signal) drawn across three schematic sheets:

| Sheet | File | Contents |
|---|---|---|
| Root | `PCB/openair-max/One-Air-Max.kicad_sch` | MCU, power distribution, sensor and CE-card interfaces |
| Battery holder | `PCB/openair-max/Battery-Holder.kicad_sch` | 18650 battery interface and BMS |
| Solar module | `PCB/openair-max/Solar-Module.kicad_sch` | Charger and solar input conditioning |

## Repository structure

```
PCB/
└── openair-max/             # Main PCB (schematic, layout, DRC rules, production/)
libraries/                   # Shared 3D model libraries
├── 3d/                      # STEP models
└── easyeda2kicad/           # easyeda2kicad WRL models
```

## Key components

| Function | Part | Ref |
|---|---|---|
| MCU / Wi-Fi / BLE | ESP32-C6-WROOM-1-N8 | U1 |
| Buck converter | TPS62902 | U2 |
| Hardware watchdog timer | TPL5010 | U3 |
| Battery management system (BMS) | BQ7790512 (BQ76905 family) | U4 |
| Battery charger (solar + USB-C) | BQ25798 | U5 |
| Boost converter | TPS62903 | U6 |
| Power mux / load switch (RC soft-start) | TPS2117 | U7 |
| I²C-to-dual-UART bridge | WK2132 | U8 |
| EEPROM (2 Mbit, optional — DNP) | AT24CM02 | U12 |
| High-side load switches (switched sensor rails) | TPS27081A | U13–U19 |

**Power architecture:** The BQ25798 charger takes two power inputs — the solar cell on **VAC1** and USB-C on **VAC2** — and charges the 3S 18650 pack behind the BQ76905 BMS. The system rail is switched through the TPS2117 with an RC soft-start network to limit inrush current, and a 470 µF bulk capacitor ahead of the load switch provides brownout immunity.

## Plug-in modules

The gas sensors and the cellular option are separate boards that plug into this board's connectors. The assembled Open Air Max (O-M-1PPSTON-CE) carries all of them:

| Module | Repository | Connector on this board |
|---|---|---|
| SUNX CO₂ sensor module (SenseAir Sunlight, UART) | [pcb-module-sunx](https://github.com/Open-Air-Foundation/pcb-module-sunx) | `CO2` — 8-pin Molex PicoBlade: switched 5 V (VOUT2, EN_CO2 = IO5), GND, UART0 (IO0/IO1) |
| SGP41 TVOC/NOₓ sensor module (I²C) | [pcb-module-sgp41](https://github.com/Open-Air-Foundation/pcb-module-sgp41) | `I2C1` / `I2C2` — 1×4 sockets, 2.54 mm: 3.3 V, GND, SCL (IO6), SDA (IO7) |
| SHT4x temperature/humidity sensor module (I²C) | [pcb-module-sht40](https://github.com/Open-Air-Foundation/pcb-module-sht40) | `I2C1` / `I2C2` — the two sockets are identical; one takes the SGP41 module, the other the SHT4x module |
| AlphaSense ADC module — NO₂ (A43F) and O₃ (A431), I²C | [pcb-module-alphasense-adc](https://github.com/Open-Air-Foundation/pcb-module-alphasense-adc) | `NO2` — 8-pin Molex PicoBlade: switched 5 V and 3.3 V (VOUT6/VOUT5, EN_NO2 = IO4), GND, I²C |
| CE card — 4G cellular (LTE Cat 1) | [pcb-module-ce-card](https://github.com/Open-Air-Foundation/pcb-module-ce-card) | `CE1` — 8-pin Molex PicoBlade: switched 5 V and 3.3 V (VOUT4/VOUT3, EN_CE = IO15), power/reset control (IO23, IO21), UART (IO16/IO17) |

The particulate sensors (2× Plantower PMS5003) plug straight into the `PM1`/`PM2` PicoBlade sockets (switched 5 V + UART through the WK2132 bridge) and have no module board of their own. `I2C3` (4-pin, 1.25 mm; I²C) and `IO1` (8-pin PicoBlade: 5 V, 3.3 V, I²C, IO18–IO20) are spare expansion connectors.

## Toolchain

- **KiCad 10** (file format 2026-02). Symbols and footprints are embedded in the design files; all 3D models are bundled in `libraries/` and referenced project-relative — the project opens without any extra library setup.
- **[Fabrication Toolkit](https://github.com/bennymeg/Fabrication-Toolkit)** plugin — generates the Gerber/BOM/CPL production set in JLCPCB-compatible format.
- Active DRC rules are in `PCB/openair-max/One-Air-Max.kicad_dru`; `PCB/openair-max/DRC/JLCPCB.kicad_dru` is a slightly stricter JLCPCB reference template.

## Fabrication

`PCB/openair-max/production/` holds the ready-to-order set for the released version: Gerber zip, BOM, designators, and pick-and-place positions (generated with Fabrication Toolkit, LCSC part numbers for assembly at JLCPCB).

## Versioning & releases

Hardware revisions are tagged on this repository. Per-version release notes — chip-level changes and the matching production file set — are published on the [GitHub Releases](../../releases) page.

## Related repositories

- [firmware-openair-max](https://github.com/Open-Air-Foundation/firmware-openair-max) — ESP32-C6 firmware for this board
- [pcb-module-sunx](https://github.com/Open-Air-Foundation/pcb-module-sunx) — SUNX CO₂ sensor module (SenseAir Sunlight)
- [pcb-module-sgp41](https://github.com/Open-Air-Foundation/pcb-module-sgp41) — SGP41 TVOC/NOₓ sensor module
- [pcb-module-sht40](https://github.com/Open-Air-Foundation/pcb-module-sht40) — SHT4x temperature/humidity sensor module
- [pcb-module-alphasense-adc](https://github.com/Open-Air-Foundation/pcb-module-alphasense-adc) — AlphaSense ADC module (NO₂ / O₃ electrochemical sensors)
- [pcb-module-ce-card](https://github.com/Open-Air-Foundation/pcb-module-ce-card) — 4G cellular extension (CE) card; the assembled Open Air Max includes this card, fitted to the CE1 connector

## License

This is open-source hardware. The design files in this repository are licensed under the
[Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/) license — see [LICENSE](LICENSE).

You are free to use, modify, and manufacture these designs, including commercially, provided you credit AirGradient and share derivative designs under the same license.

## Maintainers

AirGradient hardware team.
