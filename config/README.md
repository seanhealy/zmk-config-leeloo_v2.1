# Development and Advanced Configuration

This directory contains the ZMK configuration files for the Leeloo v2.1 split keyboard.

## File Structure

```
config/
├── leeloo.keymap              # Main keymap (shared between BLE and Dongle modes)
├── leeloo.conf                # Basic keyboard configuration
└── boards/shields/leeloo_rev2/
    ├── Kconfig.shield         # Shield definitions
    ├── Kconfig.defconfig      # Default configurations
    ├── leeloo_rev2_dongle_xiao.overlay  # Dongle hardware mapping
    ├── leeloo_rev2_dongle_xiao.conf     # Dongle-specific settings
    └── leeloo_rev2_dongle_xiao.keymap   # Dongle keymap (includes main keymap)
```

## Keymap Customization

### Main Keymap File

Edit `leeloo_base.keymap` to customize your key bindings. This file is automatically used by both BLE and Dongle modes, ensuring consistency across connection methods.

## Advanced Configuration

### Dongle Mode Details

The dongle implementation uses:

- **Hardware**: Seeed XIAO BLE board
- **Role**: Central device (connects both halves and routes to USB)
- **Matrix**: Full 5×12 Leeloo matrix mapping
- **Power**: Optimized for always-on USB operation

### BLE Mode Details

- **Left Half**: Acts as central, connects to computer via Bluetooth
- **Right Half**: Peripheral, connects to left half
- **Profiles**: 5 Bluetooth profiles available for device switching

### Build Artifacts

The build system generates clearly named firmware files:

**Dongle Mode:**

- `dongle_leeloo_xiao-*.uf2` → XIAO BLE dongle
- `dongle_leeloo_left_peripheral-*.uf2` → Left half (peripheral)
- `dongle_leeloo_right_peripheral-*.uf2` → Right half (peripheral)

**BLE Mode:**

- `ble_leeloo_left_central-*.uf2` → Left half (central)
- `ble_leeloo_right_peripheral-*.uf2` → Right half (peripheral)

### Power Management

#### Dongle Mode Benefits

- Both keyboard halves run as peripherals (better battery life)
- No need to maintain connection between halves and computer
- USB power for dongle eliminates battery concerns

#### BLE Mode Configuration

- Optimized for direct Bluetooth connection
- Automatic power management
- Sleep modes for extended battery life

## Development Setup

### Local Development

If you want to build locally instead of using GitHub Actions:

1. Install ZMK development environment: [ZMK Setup Guide](https://zmk.dev/docs/development/setup)

2. Build commands:

   ```bash
   # For Dongle Mode
   west build -d build/dongle -b seeeduino_xiao_ble -- -DSHIELD=leeloo_rev2_dongle_xiao
   west build -d build/left_p -b nice_nano_v2 -- -DSHIELD="leeloo_rev2_left nice_view_adapter nice_view" -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n
   west build -d build/right_p -b nice_nano_v2 -- -DSHIELD="leeloo_rev2_right nice_view_adapter nice_view" -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

   # For BLE Mode
   west build -d build/left_c -b nice_nano_v2 -- -DSHIELD="leeloo_rev2_left nice_view_adapter nice_view"
   west build -d build/right_p -b nice_nano_v2 -- -DSHIELD="leeloo_rev2_right nice_view_adapter nice_view"
   ```
