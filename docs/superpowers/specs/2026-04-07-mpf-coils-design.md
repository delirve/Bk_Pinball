# MPF FAST 3208 Coil Configuration Design

**Date:** 2026-04-07
**Project:** Bk_Pinball (Banjo-Kazooie Pinball)

## Overview

Configure the FAST Pinball 3208 board coils, flippers, and slingshots in MPF. The board has 8 outputs (pins 0–7), all allocated. Platform changes from `virtual` to `fast` connected on COM4.

## Hardware

- **Board:** FAST Pinball 3208
- **Connection:** COM4
- **Node:** 0 (default)
- **Outputs used:** 8 of 8

| Pin | Name | MPF Number |
|-----|------|------------|
| 0 | Left flipper power (main) | 0-0 |
| 1 | Left flipper hold | 0-1 |
| 2 | Right flipper power (main) | 0-2 |
| 3 | Right flipper hold | 0-3 |
| 4 | Upper flipper power (main) | 0-4 |
| 5 | Upper flipper hold | 0-5 |
| 6 | Right slingshot | 0-6 |
| 7 | Left slingshot | 0-7 |

> `c_trough_eject` remains a placeholder with no number — requires a second board, to be addressed later.

## File Changes

### 1. `config/config.yaml`
- Replace `config_coils.yaml` with `config_devices.yaml` in the `config:` list
- Add `hardware:` section with `platform: fast`
- Add `fast:` section with `ports: com4`

### 2. `config/config_coils.yaml` → replaced by `config/config_devices.yaml`
Single file containing all device definitions:
- **Coils** — 8 hardware coils + trough eject placeholder
- **Flippers** — 3 dual-wound flippers (left, right, upper), each with EOS switch
- **Autofire coils** — 2 slingshots

### 3. `config/config_switches.yaml`
Add 6 new switches (hardware numbers left blank to fill in later):
- `s_flipper_upper` — upper flipper button
- `s_slingshot_left` — left slingshot
- `s_slingshot_right` — right slingshot
- `s_flipper_left_eos` — left flipper end-of-stroke
- `s_flipper_right_eos` — right flipper end-of-stroke
- `s_flipper_upper_eos` — upper flipper end-of-stroke

## Flipper EOS Behavior

Each flipper is configured with an `eos_switch`. When the flipper reaches full stroke and trips the EOS switch, MPF cuts power to the main coil and relies on the hold coil to keep the flipper up. This prevents heat damage to the hold winding.

## Out of Scope

- Switch hardware number assignment (to be done when wiring is complete)
- Trough eject coil (requires second board)
- Pulse timing tuning (default values used; tune during physical testing)
