# Leeloo MK2 Dongle Implementation Summary

## What Was Implemented

This implementation adds dongle support to your Leeloo MK2 split keyboard. The dongle (XIAO BLE) acts as the central device, with both keyboard halves becoming peripherals.

## Files Created

### 1. Shield Configuration Files
- `config/boards/shields/leeloo_rev2/Kconfig.shield` - Defines the dongle shield
- `config/boards/shields/leeloo_rev2/Kconfig.defconfig` - Configures dongle settings
- `config/boards/shields/leeloo_rev2/leeloo_rev2_dongle_xiao.overlay` - Hardware mapping

### 2. Build Configuration
- Updated `build.yaml` to include dongle build and convert keyboard halves to peripherals

## Key Configuration Details

### Dongle Settings
- **Keyboard Name**: "Leeloo Dongle" 
- **Role**: Central device
- **Peripherals**: 2 (left and right halves)
- **BT Connections**: 7 (2 peripherals + 5 BT profiles)
- **Hardware**: XIAO BLE (seeeduino_xiao_ble)

### Keyboard Halves
- **Role**: Converted to peripherals
- **Displays**: nice!view displays will continue working
- **Hardware**: Existing nice_nano_v2 boards (unchanged)

## How It Works

1. **XIAO BLE Dongle**: Acts as central, connects to computer via USB
2. **Left Half**: Connects to dongle via Bluetooth as peripheral
3. **Right Half**: Connects to dongle via Bluetooth as peripheral
4. **Key Presses**: Route from halves → dongle → computer via USB

## Build Targets Created

The build system will now generate:
- `leeloo_rev2_dongle_xiao-seeeduino_xiao_ble-zmk.uf2` - Dongle firmware
- `leeloo_rev2_left-nice_nano_v2-zmk.uf2` - Left half (peripheral)
- `leeloo_rev2_right-nice_nano_v2-zmk.uf2` - Right half (peripheral)
- Settings reset firmware for both board types

## Setup Process

### 1. Flash Settings Reset
Flash settings_reset firmware to all three devices:
- Both keyboard halves (nice_nano_v2)
- XIAO BLE dongle

### 2. Flash New Firmware
- Flash dongle firmware to XIAO BLE
- Flash peripheral firmware to both keyboard halves

### 3. Pairing
Reset all three devices simultaneously to initiate pairing between dongle and halves.

### 4. Connect to Computer
Connect the XIAO BLE dongle to your computer via USB-C.

## Benefits

- **Better Battery Life**: Both keyboard halves will last longer as peripherals
- **Easier Connection**: USB connection instead of Bluetooth pairing with computer
- **Multiple Profiles**: Switch between different computers using dongle's BT profiles
- **Lower Average Latency**: Improved performance for most usage patterns

## Rollback Process

To revert to direct Bluetooth connection:
1. Flash settings_reset to all devices
2. Remove cmake-args from build.yaml for the keyboard halves
3. Remove dongle entries from build.yaml
4. Flash original firmware to keyboard halves

## Hardware Required

- **XIAO BLE board** - for the dongle (no display needed)
- **USB-C cable** - to connect dongle to computer
- **Existing keyboard halves** - no changes needed

## Matrix Layout

The dongle uses the exact same 5x12 matrix layout as your current Leeloo configuration:
- 60 keys total across both halves
- Maintains all encoder positions
- Preserves thumb cluster layouts
- Compatible with your existing keymap

Your `config/leeloo.keymap` file will work without any changes.
