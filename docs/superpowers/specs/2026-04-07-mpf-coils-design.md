# MPF FAST 3208 Coil Configuration Design

**Date:** 2026-04-07
**Project:** Bk_Pinball (Banjo-Kazooie Pinball)

## Overview

Configure the FAST Pinball 3208 board coils, flippers, and slingshots in MPF. The board has 8 outputs (pins 0–7), all allocated. Platform switches from `virtual` to `fast` on COM4. Switch hardware numbers are left blank — fill them in before the first hardware run.

## Hardware

- **Board:** FAST Pinball 3208
- **Connection:** COM4 (uppercase on Windows)
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

## File Changes

> **Step order matters:** Follow steps 1–5 in order. Create the new file before updating
> `config.yaml`, and do not delete the old file until `config.yaml` already points to the new one.
> All 5 steps must be completed before attempting to run MPF.

### Step 1 — Create `config/config_coils_flippers.yaml`

> Do not confuse with the existing `config_ball_devices.yaml`.

```yaml
#config_version=6

coils:
  c_flipper_left_main:
    number: 0-0
    default_pulse_ms: 30
  c_flipper_left_hold:
    number: 0-1
    allow_enable: true
  c_flipper_right_main:
    number: 0-2
    default_pulse_ms: 30
  c_flipper_right_hold:
    number: 0-3
    allow_enable: true
  c_flipper_upper_main:
    number: 0-4
    default_pulse_ms: 30
  c_flipper_upper_hold:
    number: 0-5
    allow_enable: true
  c_slingshot_right:
    number: 0-6
    default_pulse_ms: 10  # placeholder; tune during hardware testing
  c_slingshot_left:
    number: 0-7
    default_pulse_ms: 10  # placeholder; tune during hardware testing
  # c_trough_eject:       # trough not installed; uncomment when wired
  #   number:

flippers:
  flipper_left:
    main_coil: c_flipper_left_main
    hold_coil: c_flipper_left_hold
    activation_switch: s_flipper_left
    eos_switch: s_flipper_left_eos
  flipper_right:
    main_coil: c_flipper_right_main
    hold_coil: c_flipper_right_hold
    activation_switch: s_flipper_right
    eos_switch: s_flipper_right_eos
  flipper_upper:
    main_coil: c_flipper_upper_main
    hold_coil: c_flipper_upper_hold
    activation_switch: s_flipper_upper
    eos_switch: s_flipper_upper_eos

autofire_coils:
  slingshot_left:
    coil: c_slingshot_left
    switch: s_slingshot_left
  slingshot_right:
    coil: c_slingshot_right
    switch: s_slingshot_right
```

### Step 2 — Update `config/config.yaml`

Replace `config_coils.yaml` with `config_coils_flippers.yaml` in the `config:` list.
Keep `config_switches.yaml` and `config_ball_devices.yaml` unchanged.
Add the `hardware:` and `fast:` blocks.

```yaml
hardware:
  platform: fast

fast:
  ports: COM4

config:
  - config_switches.yaml
  - config_ball_devices.yaml
  - config_coils_flippers.yaml
```

### Step 3 — Delete `config/config_coils.yaml`

Delete the old file. Its contents have moved to `config_coils_flippers.yaml`.

### Step 4 — Update `config/config_switches.yaml`

Add 6 new switches. All switch `number:` values (existing and new) are left blank — FAST
switch numbers use `node-pin` format (e.g., `0-0`). Fill them in before the first hardware run.

```yaml
s_flipper_upper:
  number:
  tags: upper_flipper
s_slingshot_left:
  number:
  tags: slingshot
s_slingshot_right:
  number:
  tags: slingshot
s_flipper_left_eos:
  number:
  tags: eos
s_flipper_right_eos:
  number:
  tags: eos
s_flipper_upper_eos:
  number:
  tags: eos
```

### Step 5 — Update `config/config_ball_devices.yaml` (required)

> This step is **required**. The old `config_coils.yaml` (which defined `c_trough_eject`)
> is being deleted. If `bd_trough` still references `eject_coil: c_trough_eject`, MPF
> will fail to load because the coil no longer exists.

Comment out `eject_coil:` and `eject_targets:` on `bd_trough`. The `ball_switches:` line
must remain — MPF requires it to track ball count.

```yaml
bd_trough:
  ball_switches: s_trough_1, s_trough_2, s_trough_3, s_trough_4
  # eject_coil: c_trough_eject      # trough not installed
  # eject_targets: bd_plunger       # trough not installed
  tags: trough, home, drain
```

## Flipper EOS Behavior

All three flippers have physical EOS switches. MPF's `eos_switch` parameter disables the
main coil when the flipper reaches full stroke, while the hold coil keeps the flipper up.

## Notes

- Switch numbers are blank across the entire `config_switches.yaml` file. MPF will not
  connect to FAST hardware until all numbers are assigned.
- Trough is not installed. `c_trough_eject`, `eject_coil:`, and `eject_targets:` are all
  commented out and must be restored when the trough is wired.
