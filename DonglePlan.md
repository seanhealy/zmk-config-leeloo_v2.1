# Leeloo Dongle Setup Plan

Reference:
https://v0-3-branch.zmk.dev/docs/development/hardware-integration/dongle

## Overview

Convert the split Leeloo keyboard to use a XIAO BLE dongle as the central device. This makes both keyboard halves peripherals that connect to the dongle, which then connects to your computer via USB.

## Step 1: Create Dongle Shield Files

Create directory: `boards/shields/leeloo_dongle/`

### `Kconfig.shield`

```
config SHIELD_LEELOO_DONGLE
  def_bool $(shields_list_contains,leeloo_dongle)
```

### `Kconfig.defconfig`

```
if SHIELD_LEELOO_DONGLE

config ZMK_KEYBOARD_NAME
  default "Leeloo Dongle"

config ZMK_SPLIT_ROLE_CENTRAL
  default y

config ZMK_SPLIT
  default y

# Set to 2 for left + right halves
config ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS
  default 2

# Set to peripherals (2) + BT profiles (5) + buffer (1)
config BT_MAX_CONN
  default 8

config BT_MAX_PAIRED
  default 8

endif
```

### `leeloo_dongle.overlay`

```
#include <dt-bindings/zmk/matrix_transform.h>

/ {
  chosen {
    zmk,kscan = &mock_kscan;
    zmk,physical-layout = &physical_layout0;
  };

  mock_kscan: mock_kscan_0 {
    compatible = "zmk,kscan-mock";
    columns = <0>;
    rows = <0>;
    events = <0>;
  };

  default_transform: keymap_transform_0 {
    compatible = "zmk,matrix-transform";
    columns = <12>;
    rows = <5>;
    // | SW1  | SW2  | SW3  | SW4  | SW5  | SW6  |                 | SW6  | SW5  | SW4  | SW3  | SW2  | SW1  |
    // | SW7  | SW8  | SW9  | SW10 | SW11 | SW12 |                 | SW12 | SW11 | SW10 | SW9  | SW8  | SW7  |
    // | SW13 | SW14 | SW15 | SW16 | SW17 | SW18 |                 | SW18 | SW17 | SW16 | SW15 | SW14 | SW13 |
    // | SW19 | SW20 | SW21 | SW22 | SW23 | SW24 | SW29 |   | SW29 | SW24 | SW23 | SW22 | SW21 | SW20 | SW19 |
    //                      | SW25 | SW26 | SW27 | SW28 |   | SW28 | SW27 | SW26 | SW25 |
    map = <
    RC(0,0) RC(0,1) RC(0,2) RC(0,3) RC(0,4) RC(0,5)                 RC(0,6) RC(0,7) RC(0,8) RC(0,9) RC(0,10) RC(0,11)
    RC(1,0) RC(1,1) RC(1,2) RC(1,3) RC(1,4) RC(1,5)                 RC(1,6) RC(1,7) RC(1,8) RC(1,9) RC(1,10) RC(1,11)
    RC(2,0) RC(2,1) RC(2,2) RC(2,3) RC(2,4) RC(2,5)                 RC(2,6) RC(2,7) RC(2,8) RC(2,9) RC(2,10) RC(2,11)
    RC(3,0) RC(3,1) RC(3,2) RC(3,3) RC(3,4) RC(3,5) RC(4,5) RC(4,6) RC(3,6) RC(3,7) RC(3,8) RC(3,9) RC(3,10) RC(3,11)
                            RC(4,1) RC(4,2) RC(4,3) RC(4,4) RC(4,7) RC(4,8) RC(4,9) RC(4,10)
    >;
  };

  physical_layout0: physical_layout_0 {
    compatible = "zmk,physical-layout";
    display-name = "Default Layout";
    transform = <&default_transform>;
  };
};
```

Matrix transform copied from official ZMK leeloo_common.dtsi.

## Step 2: Update build.yaml

Replace your current `build.yaml` content with:

```yaml
include:
   # Dongle (central)
   - board: xiao_ble
     shield: leeloo_dongle

   # Left half (peripheral) with nice!view
   - board: nice_nano_v2
     shield: leeloo_rev2_left nice_view_adapter nice_view
     cmake-args: -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

   # Right half (peripheral) with nice!view
   - board: nice_nano_v2
     shield: leeloo_rev2_right nice_view_adapter nice_view
     cmake-args: -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

   # Settings reset firmwares
   - board: nice_nano_v2
     shield: settings_reset
   - board: xiao_ble
     shield: settings_reset
```

---

## Step 3: Flash Firmware

⚠️ **IMPORTANT:** Flash settings reset on all devices first!

### 3a. Settings Reset (Do This First)

For each device:

1. **Enter bootloader**: Double-tap reset button
   - nice!nano v2 → appears as `NICENANO` drive
   - XIAO BLE → appears as `XIAO-SENSE` drive
2. **Flash**: Drag `settings_reset-<board>-zmk.uf2` to the drive
3. **Wait**: Drive disappears, device reboots

### 3b. Flash New Firmware

**Left half:** Double-tap reset → drag `leeloo_rev2_left_nice_view_adapter_nice_view-nice_nano_v2-zmk.uf2` to `NICENANO` left.

**Right half:** Double-tap reset → drag `leeloo_rev2_right_nice_view_adapter_nice_view-nice_nano_v2-zmk.uf2` to `NICENANO` right.

**Dongle:** Double-tap reset → drag `leeloo_dongle-xiao_ble-zmk.uf2` to `XIAO-SENSE`

### 3c. Test Connection

Connect dongle to computer. Should auto-pair with both halves and allow typing.

**If pairing fails:** Repeat settings reset on all devices.

---

## Step 4: Keymap Changes

- **Keymap changes:** Only need to reflash the dongle
