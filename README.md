# Leeloo v2.1 ZMK Configuration

![Leeloo v2.1](https://github.com/ClicketySplit/build-guides/blob/main/leeloo/images/gallery/Leeloo-v2-ZMK.jpg)

**Keyboard Designer**: [clicketysplit.ca](https://clicketysplit.ca)  
**GitHub**: [ClicketySplit](https://github.com/ClicketySplit)  
**Hardware Supported**: nice!nano v2 with OLED/nice!view displays

## What's This Repository?

This is a complete ZMK configuration for the Leeloo v2.1 split keyboard that supports **two connection modes**:

1. **🔵 Direct Bluetooth (BLE)** - Traditional wireless split keyboard
2. **🔌 USB Dongle Mode** - Use a USB dongle for wired connection with wireless halves

## Quick Start

### 1. Fork This Repository

Click "Fork" on GitHub and enable GitHub Actions in your fork.

### 2. Choose Your Connection Mode

**🔵 For Direct Bluetooth (BLE):**

- Connect each half directly to your computer via Bluetooth
- Better for single-device use
- Standard split keyboard experience

**🔌 For USB Dongle Mode:**

- Both halves connect to a small USB dongle (XIAO BLE)
- Dongle plugs into your computer via USB-C
- Better battery life, more reliable connection
- Easy switching between multiple computers

### 3. Get Your Firmware

GitHub Actions automatically builds firmware when you push changes. Download from the "Actions" tab:

**For BLE Mode:**

- `ble_leeloo_left_central-*.uf2` → Left half (acts as central)
- `ble_leeloo_right_peripheral-*.uf2` → Right half

**For Dongle Mode:**

- `dongle_leeloo_xiao-*.uf2` → XIAO BLE dongle
- `dongle_leeloo_left_peripheral-*.uf2` → Left half
- `dongle_leeloo_right_peripheral-*.uf2` → Right half

### 4. Flash Firmware

1. **Reset settings first** (recommended): Flash `settings_reset_*.uf2` to all devices
2. **Flash the firmware**: Drag and drop the `.uf2` files to each device when in bootloader mode
3. **Pair devices**: Reset all devices simultaneously to initiate pairing

## Hardware Requirements

### For Both Modes

- 2x nice!nano v2 controllers
- 2x nice!view displays (recommended)
- Batteries for each half

### Additional for Dongle Mode

- 1x Seeed XIAO BLE board
- USB-C cable

## Customizing Your Keymap

### Easy Way: ZMK Studio (Recommended)

1. Open [ZMK Studio](https://zmk.dev/zmk-studio/) in your browser
2. Load your `.uf2` firmware file
3. Make changes visually
4. Apply changes directly to your keyboard

### Advanced Way: Code Editing

Edit `config/leeloo.keymap` and push to GitHub. Your changes will automatically apply to both BLE and Dongle modes.

Key files:

- `config/leeloo.keymap` - Your keymap (shared between both modes)
- `build.yaml` - Build configuration (don't change unless you know what you're doing)

## Features

### Leeloo v2.1 Hardware

- 4×6 + 5 thumb keys per half (60 keys total)
- Kailh Choc low-profile switches (18mm spacing)
- Socketed switches and rotary encoders
- Native OLED/nice!view display support
- Support for 110mAh, 600mAh, or 700mAh batteries
- Per-switch RGB underglow support
- ZMK v3.5 Power Domains (minimal battery drain)

### ZMK Firmware Features

- 5 Bluetooth profiles per device
- Rotary encoder support
- RGB underglow (if installed)
- Low power consumption
- Real-time configuration with ZMK Studio

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

### Display Issues

If nice!view displays aren't working, ensure this line is uncommented in your keymap:

```c
nice_view_spi: &spi0 { cs-gpios = <&pro_micro 4 GPIO_ACTIVE_HIGH>; };
```

## Support

For hardware questions: [Contact Clickety Split](https://clicketysplit.ca/pages/contact-us)

For ZMK firmware questions: [ZMK Documentation](https://zmk.dev/docs/)

---

**Clickety Split - For the love of split keyboards** ❤️
