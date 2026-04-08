# MPF FAST 3208 Coil Configuration Design

**Date:** 2026-04-07
**Project:** Bk_Pinball (Banjo-Kazooie Pinball)

## Overview

Configure the FAST Pinball 3208 board coils, flippers, and slingshots in MPF. The board has 8 outputs (pins 0–7), all allocated.

This task is split into two phases:

- **Phase 1 (this task):** Add coil hardware numbers, flipper/slingshot device definitions, and new switch placeholders. Platform remains `virtual` so MPF can still run without real hardware.
- **Phase 2 (separate task):** Assign hardware numbers to all switches, switch platform to `fast`, and restore the trough eject coil on a second board.

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

## Phase 1 File Changes

> **Step order matters:** Follow steps 1–5 in order. Create the new file before updating
> `config.yaml`, and do not delete the old file until `config.yaml` already points to the new one.
> All 5 steps must be completed before attempting to run MPF — the config is in an intermediate
> state until all steps are done (e.g., `flipper_upper` references `s_flipper_upper` which is
> added in Step 4, and flippers reference EOS switches also added in Step 4).

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
  # c_trough_eject:       # deferred to Phase 2 (requires second board)
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
Do **not** add `hardware:` or `fast:` blocks — platform stays `virtual`.

```yaml
config:
  - config_switches.yaml
  - config_ball_devices.yaml
  - config_coils_flippers.yaml
```

### Step 3 — Delete `config/config_coils.yaml`

Delete the old file. Its contents have moved to `config_coils_flippers.yaml`.

### Step 4 — Update `config/config_switches.yaml`

Add 6 new switches with blank `number:` values (consistent with existing switches).
Numbers are filled in during Phase 2. FAST switch numbers use `node-pin` format (e.g., `0-0`).

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
> will fail to load because the coil no longer exists in any loaded file.

Comment out the `eject_coil:` and `eject_targets:` lines on `bd_trough`. The device
itself (including `ball_switches:`) must remain — MPF requires a trough device with
`ball_switches` to track ball count.

## Flipper EOS Behavior

MPF's `flippers:` section supports an `eos_switch` parameter for dual-wound flippers.
When the EOS switch activates (flipper reaches full stroke), MPF disables the main coil
while the hold coil remains energized, preventing heat damage.

> **Hardware assumption:** EOS switches are assumed to be physically present and wired on
> all three flippers. If a flipper has no EOS switch, remove the `eos_switch:` line for
> that flipper — otherwise MPF will hold the main coil energized indefinitely (the EOS
> condition never fires), which can burn out the winding.
>
> **Virtual platform note:** In virtual mode, EOS switches with blank `number:` values are
> accepted by MPF as switches that never activate. If MPF raises a load error for the blank
> EOS switch numbers, temporarily comment out the `eos_switch:` lines in the flipper
> definitions until Phase 2 when real numbers are assigned.

## Phase 2 Checklist (Deferred)

When hardware wiring is complete:

1. Assign real numbers to all blank `number:` entries in `config_switches.yaml`
2. Add to `config.yaml` (verify exact syntax against MPF v0.80 docs — `hardware: platform:` nesting may differ from earlier versions):
   ```yaml
   hardware:
     platform: fast
   fast:
     ports: COM4
   ```
3. Wire second board; uncomment `c_trough_eject` in `config_coils_flippers.yaml`
4. Restore `eject_coil:` and `eject_targets:` on `bd_trough` in `config_ball_devices.yaml`

## Out of Scope for Phase 1

- Switch hardware number assignment
- Platform switch from virtual to fast
- Trough eject coil (requires second board)
- Pulse timing tuning (defaults used; tune during physical testing)
