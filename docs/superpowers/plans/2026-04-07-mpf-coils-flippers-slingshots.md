# MPF Coils, Flippers & Slingshots Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Configure FAST Pinball 3208 board coils, flippers (with EOS), and slingshots in MPF config files.

**Architecture:** Pure YAML config changes across 5 files. A new `config_coils_flippers.yaml` replaces `config_coils.yaml` and adds flipper and autofire coil device definitions. Platform switches from `virtual` to `fast` on COM4. Switch numbers stay blank until hardware is wired.

**Tech Stack:** MPF (Mission Pinball Framework) v0.80.0.dev12, YAML config, FAST Pinball 3208 board

**Spec:** `docs/superpowers/specs/2026-04-07-mpf-coils-design.md`

---

## Chunk 1: Create device config file and wire it into main config

### Task 1: Create `config/config_coils_flippers.yaml`

**Files:**
- Create: `config/config_coils_flippers.yaml`

> **Note:** There is no traditional test suite for MPF YAML config. Verification is done by
> running MPF and checking that it loads without config errors. Hardware connection errors
> (COM4 not found) are expected and OK — config errors are not.

- [ ] **Step 1.1: Create the file**

Create `config/config_coils_flippers.yaml` with the following content exactly:

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
    default_pulse_ms: 10
  c_slingshot_left:
    number: 0-7
    default_pulse_ms: 10
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

- [ ] **Step 1.2: Commit**

```bash
git add config/config_coils_flippers.yaml
git commit -m "feat: add coils, flippers, and slingshot config for FAST 3208"
```

---

### Task 2: Update `config/config.yaml`

**Files:**
- Modify: `config/config.yaml`

- [ ] **Step 2.1: Replace the config import and add hardware platform**

The current file looks like:

```yaml
#config_version=6

config:
  - config_switches.yaml
  - config_ball_devices.yaml
  - config_coils.yaml
modes:
  - attract
  - base

playfields:
  playfield:
    default_source_device: bd_trough
    tags: default
```

Replace with:

```yaml
#config_version=6

hardware:
  platform: fast

fast:
  ports: COM4

config:
  - config_switches.yaml
  - config_ball_devices.yaml
  - config_coils_flippers.yaml
modes:
  - attract
  - base

playfields:
  playfield:
    default_source_device: bd_trough
    tags: default
```

- [ ] **Step 2.2: Commit**

```bash
git add config/config.yaml
git commit -m "feat: switch MPF platform to FAST on COM4"
```

---

### Task 3: Delete old coils file

**Files:**
- Delete: `config/config_coils.yaml`

- [ ] **Step 3.1: Delete the file**

```bash
git rm config/config_coils.yaml
```

- [ ] **Step 3.2: Commit**

```bash
git commit -m "chore: remove config_coils.yaml (replaced by config_coils_flippers.yaml)"
```

---

## Chunk 2: Add new switches and fix ball device

### Task 4: Add new switches to `config/config_switches.yaml`

**Files:**
- Modify: `config/config_switches.yaml`

The current file is:

```yaml
#config_version=6

switches:
  #Cabinet Switches
  s_flipper_left:
    number:
    tags: left_flipper
  s_flipper_right:
    number:
    tags: right_flipper
  # Trough Switches
  s_trough_1:
    number:
  s_trough_2:
    number:
  s_trough_3:
    number:
  s_trough_4:
    number:
  s_plunger_lane:
    number:
```

- [ ] **Step 4.1: Add the 6 new switches**

Append the following under the existing switches (inside the `switches:` block):

```yaml
  # Upper Flipper
  s_flipper_upper:
    number:
    tags: upper_flipper
  # Slingshots
  s_slingshot_left:
    number:
    tags: slingshot
  s_slingshot_right:
    number:
    tags: slingshot
  # EOS Switches
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

> All `number:` values are intentionally blank. FAST switch numbers use `node-pin` format
> (e.g., `0-0`). Fill them in when the switches are physically wired.

- [ ] **Step 4.2: Commit**

```bash
git add config/config_switches.yaml
git commit -m "feat: add upper flipper, slingshot, and EOS switch placeholders"
```

---

### Task 5: Comment out trough eject in `config/config_ball_devices.yaml`

**Files:**
- Modify: `config/config_ball_devices.yaml`

> **This step is required.** `config_coils.yaml` (which defined `c_trough_eject`) was
> deleted in Task 3. If `bd_trough` still references `eject_coil: c_trough_eject`, MPF
> will fail at load time with an unknown coil error.

The current `bd_trough` entry looks like:

```yaml
  bd_trough:
    ball_switches: s_trough_1, s_trough_2, s_trough_3, s_trough_4
    eject_coil: c_trough_eject
    eject_targets: bd_plunger
    tags: trough, home, drain
```

- [ ] **Step 5.1: Comment out the eject lines**

Replace with:

```yaml
  bd_trough:
    ball_switches: s_trough_1, s_trough_2, s_trough_3, s_trough_4
    # eject_coil: c_trough_eject      # trough not installed
    # eject_targets: bd_plunger       # trough not installed
    tags: trough, home, drain
```

- [ ] **Step 5.2: Commit**

```bash
git add config/config_ball_devices.yaml
git commit -m "chore: comment out trough eject (hardware not installed)"
```

---

## Chunk 3: Verification

### Task 6: Verify config loads without YAML errors

- [ ] **Step 6.1: Run MPF and check for config errors**

From the project root (`C:\Users\Nicho\Documents\Bk_Pinball`), run MPF. It will fail to
connect to the FAST board on COM4 — that is expected. What you are checking for is that
MPF loads and parses all YAML successfully before it hits the hardware connection step.

```bash
cd C:\Users\Nicho\Documents\Bk_Pinball && python -m mpf -c config.yaml -v 2>&1 | head -60
```

Or if you have MPF installed in a venv (based on project logs, it's at `C:\Users\Nicho\mpfBK`):

```bash
C:\Users\Nicho\mpfBK\Scripts\python.exe -m mpf C:\Users\Nicho\Documents\Bk_Pinball -c config.yaml -v 2>&1 | head -80
```

- [ ] **Step 6.2: Confirm expected output**

You should see log lines like:
```
INFO : ConfigProcessor : Loading config: config.yaml
INFO : ConfigProcessor : Loading config: config_switches.yaml
INFO : ConfigProcessor : Loading config: config_ball_devices.yaml
INFO : ConfigProcessor : Loading config: config_coils_flippers.yaml
```

After that, MPF will attempt to connect to the FAST board and fail with a serial/COM port
error. That is fine. If you see `ERROR : ConfigProcessor` or `KeyError` or `yaml.scanner`
errors before the hardware connection attempt, there is a YAML problem to fix.

- [ ] **Step 6.3: Final commit if verification passes**

```bash
git add -A
git status  # confirm nothing unexpected is staged
git commit -m "chore: verify MPF config loads clean with FAST platform"
```

---

## When Switch Numbers Are Known

When the machine is wired, fill in the blank `number:` values in `config/config_switches.yaml`.
All entries use `node-pin` format where node is the board address (default `0`) and pin is
the switch input number on that board (e.g., `0-0`, `0-1`, `0-2`...).

When the trough is installed:
1. Uncomment `c_trough_eject` in `config/config_coils_flippers.yaml` and assign its number
2. Restore `eject_coil: c_trough_eject` and `eject_targets: bd_plunger` in `config/config_ball_devices.yaml`
