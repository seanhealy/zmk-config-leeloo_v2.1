## Plan: move **Leeloo v2.1 (nice!nano v2 halves)** to a **XIAO BLE USB dongle (central)**

### What changes conceptually

- The **XIAO BLE dongle becomes the split "central"** (it runs the keymap and talks to the host over USB/BLE).[^1]
- Both Leeloo halves become **split "peripherals"** (they scan keys and send events to the central).[^1]
- You must set the central's peripheral count so **both halves can connect** (usually `2`).[^3][^6]
- The keyboard is effectively **dependent on the dongle** in this mode (if the dongle isn't present, it won't work).[^3]

---

## 1) Add one new config file for peripheral halves

Create `config/peripheral.conf` in the config folder (next to `leeloo.conf`):

```ini
# Both halves are peripherals in dongle mode
CONFIG_ZMK_SPLIT=y
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

# Show a split-peripheral connection icon on nice!view (peripheral-only widget)
CONFIG_ZMK_WIDGET_PERIPHERAL_STATUS=y
```

The peripheral status widget is specifically meant to show split peripheral connection state and only applies on peripherals (`!ZMK_SPLIT_ROLE_CENTRAL`).[^4] This will help you see when each half is connected to the dongle.

---

## 2) Add a "dongle shield" for the XIAO BLE central

Add this tree to your repo:

```text
boards/
  shields/
    leeloo_dongle/
      Kconfig.shield
      leeloo_dongle.conf
      leeloo_dongle.overlay
```

Use these contents:

**`boards/shields/leeloo_dongle/Kconfig.shield`**

```kconfig
config SHIELD_LEELOO_DONGLE
  def_bool $(shields_list_contains,leeloo_dongle)
```

**`boards/shields/leeloo_dongle/leeloo_dongle.conf`**

```ini
# Dongle is the central
CONFIG_ZMK_SPLIT=y
CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y

# Must match number of peripherals (left + right halves = 2)
CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=2

# Set keyboard name (simpler than using Kconfig.defconfig)
CONFIG_ZMK_KEYBOARD_NAME="Leeloo Dongle"
```

That peripheral count setting is required for dongle mode so the central can connect to both halves.[^3][^6]

**`boards/shields/leeloo_dongle/leeloo_dongle.overlay`** (no key scanning on dongle)

```dts
/ {
  chosen { zmk,kscan = &kscan0; };

  kscan0: kscan0 {
    compatible = "zmk,kscan-noop";
  };
};
```

**Note**: ZMK automatically finds and uses your existing `config/leeloo.keymap` file - no need to explicitly reference it in the overlay.

---

## 3) Update build.yaml for automatic GitHub Actions builds

Update your `build.yaml` to include all firmware builds:

```yaml
# This file generates the GitHub Actions matrix
include:
   - board: nice_nano_v2
     shield: leeloo_rev2_left
   - board: nice_nano_v2
     shield: leeloo_rev2_right
   - board: xiao_ble
     shield: leeloo_dongle
   - board: nice_nano_v2
     shield: settings_reset
   - board: xiao_ble
     shield: settings_reset
```

This will automatically build:

- Left half peripheral firmware
- Right half peripheral firmware
- XIAO BLE dongle central firmware
- Settings reset firmware for both board types (for troubleshooting)

---

## 4) Build 3 firmwares (left, right, dongle)

### Left half (nice!nano v2 peripheral)

```sh
west build -d build/left -p -b nice_nano_v2 -- \
  -DSHIELD=leeloo_rev2_left \
  -DEXTRA_CONF_FILE=../config/peripheral.conf
```

### Right half (nice!nano v2 peripheral)

```sh
west build -d build/right -p -b nice_nano_v2 -- \
  -DSHIELD=leeloo_rev2_right \
  -DEXTRA_CONF_FILE=../config/peripheral.conf
```

### Dongle (XIAO BLE central)

```sh
west build -d build/dongle -p -b xiao_ble -- \
  -DSHIELD=leeloo_dongle
```

---

## 5) Flash order

1. Flash **left** (peripheral build)
2. Flash **right** (peripheral build)
3. Flash **XIAO BLE** (dongle/central build)

---

## 6) Pairing / bonds (do this early if anything is flaky)

- If connections are weird, build/flash a **settings reset** firmware for each device as part of troubleshooting; ZMK explicitly recommends this flow for connection issues.[^2]
- Remember: Bluetooth bond keys are stored on-device and can persist across reflashes, so clearing bonds can matter when you change roles/topology.[^5]

**Settings reset build commands:**

For nice!nano v2 (both halves):

```sh
west build -p -b nice_nano_v2 -- -DSHIELD=settings_reset
```

For XIAO BLE (dongle):

```sh
west build -p -b xiao_ble -- -DSHIELD=settings_reset
```

---

## 7) Keymap in dongle mode (important workflow change)

- The **keymap is effectively "applied" on the dongle**, because the split central is what runs keymap logic and turns events into HID for the host.[^1]
- Your existing `config/leeloo.keymap` will be used by the dongle shield
- Result: for most layout tweaks, you'll usually only need to **rebuild/reflash the dongle** (not both halves).
- The peripheral halves will continue showing their nice!view displays with the new peripheral status widget

---

## 8) Display behavior in dongle mode

- **Peripheral halves**: Will show normal display info plus a connection status icon indicating whether they're connected to the dongle
- **Dongle**: XIAO BLE has no built-in display, so no display considerations needed
- Your existing display configuration in `config/leeloo.conf` will continue working on the peripherals

---

[^1]: [Split Keyboards | ZMK Firmware](https://zmk.dev/docs/features/split-keyboards) (27%)

[^2]: [Troubleshooting wireless connection issues of ZMK devices.](https://zmk.dev/docs/troubleshooting/connection-issues) (26%)

[^3]: [Keyboard Dongle](https://zmk.dev/docs/development/hardware-integration/dongle) (16%)

[^4]: [zmk/app/src/display/widgets/Kconfig at main · zmkfirmware/zmk](https://github.com/zmkfirmware/zmk/blob/main/app/src/display/widgets/Kconfig) (12%)

[^5]: [Bluetooth Behavior](https://zmk.dev/docs/keymaps/behaviors/bluetooth) (11%)

[^6]: [Split Configuration | ZMK Firmware](https://zmk.dev/docs/config/split) (8%)
