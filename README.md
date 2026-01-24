# Clickety Split | Leeloo v2.1

![Leeloo v2.1](https://github.com/ClicketySplit/build-guides/blob/main/leeloo/images/gallery/Leeloo-v2-ZMK.jpg)

Keyboard Designer: [clicketysplit.ca](https://clicketysplit.ca) \
GitHub: [ClicketySplit](https://github.com/ClicketySplit)

## Leeloo v2.1 ZMK Configuration

![Leeloo v2.1](https://github.com/ClicketySplit/build-guides/blob/main/leeloo/images/gallery/Leeloo-v2-ZMK.jpg)

**Keyboard Designer**: [clicketysplit.ca](https://clicketysplit.ca)  
**GitHub**: [ClicketySplit](https://github.com/ClicketySplit)  
**Hardware Supported**: nice!nano v2 with OLED/nice!view displays

## What's This Repository?

This is a complete ZMK configuration for the Leeloo v2.1 split keyboard that supports **two connection modes**:

1. **🔵 Direct Bluetooth (BLE)** - Traditional wireless split keyboard
2. **🔌 USB Dongle Mode** - Use a USB dongle for wired connection with wireless halves

## Quick Start

### 1. Choose Your Connection Mode

**🔵 For Direct Bluetooth (BLE):**

- Connect each half directly to your computer via Bluetooth
- Better for single-device use
- Standard split keyboard experience

**🔌 For USB Dongle Mode:**

- Both halves connect to a small USB dongle (XIAO BLE)
- Dongle plugs into your computer via USB-C
- Better battery life, more reliable connection
- Easy switching between multiple computers

### 2. Get Your Firmware

GitHub Actions automatically builds firmware when you push changes. Download from the "Actions" tab:

**For BLE Mode:**

- `ble_leeloo_left_central-*.uf2` → Left half (acts as central)
- `ble_leeloo_right_peripheral-*.uf2` → Right half

**For Dongle Mode:**

- `dongle_leeloo_xiao-*.uf2` → XIAO BLE dongle
- `dongle_leeloo_left_peripheral-*.uf2` → Left half
- `dongle_leeloo_right_peripheral-*.uf2` → Right half

### 3. Flash Firmware

1. **Reset settings first** (recommended): Flash `settings_reset_*.uf2` to all devices
2. **Flash the firmware**: Drag and drop the `.uf2` files to each device when in bootloader mode
3. **Pair devices**: Reset all devices simultaneously to initiate pairing

## Hardware Requirements

### For Both Modes

- Leeloo v2.1 hardware

### Additional for Dongle Mode

- 1x Seeed XIAO BLE board
- USB-C cable

## Customizing Your Keymap

Edit `config/leeloo_base.keymap` and push to GitHub. Your changes will automatically apply to both BLE and Dongle modes.

## Features

### Leeloo v2.1 Hardware

- 4×6 + 5 thumb keys per half (60 keys total)
- Kailh Choc low-profile switches (18mm spacing)
- Socketed switches and rotary encoders
- Native OLED/nice!view display support
- Support for 110mAh, 600mAh, or 700mAh batteries

### ZMK Firmware Features

- 5 Bluetooth profiles per device
- Rotary encoder support
- RGB underglow (if installed)
- Low power consumption

## Switching Between Modes

You can switch between BLE and Dongle modes anytime:

1. Flash `settings_reset_*.uf2` to all devices
2. Flash the firmware for your desired mode
3. Reset devices to initiate pairing

## Troubleshooting

### Connection Issues

1. Flash settings reset to all devices
2. Reset all devices simultaneously
3. Re-flash firmware if needed

### Build Issues

- Check that your GitHub Actions are enabled
- Verify syntax in `config/leeloo.keymap` if you made changes
- Check the Actions tab for build errors

## Support

For hardware questions: [Contact Clickety Split](https://clicketysplit.ca/pages/contact-us)

For ZMK firmware questions: [ZMK Documentation](https://zmk.dev/docs/)

---

**Clickety Split - For the love of split keyboards** ❤️
