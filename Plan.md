# Leeloo MK2 Dongle Implementation Plan

## Overview

This plan outlines the steps needed to add dongle support to the Leeloo MK2 split keyboard using ZMK's dongle mode. The dongle will act as the central device, with both keyboard halves becoming peripherals that connect to the XIAO BLE dongle, which then connects to the computer via USB.

## Current State Analysis

- **Current Setup**: Split Leeloo Rev2 with left half as central, right half as peripheral
- **Board**: nice_nano_v2 for both halves
- **Displays**: nice_view displays on both halves
- **Target Dongle Board**: XIAO BLE (seeeduino_xiao_ble)
- **Reference Implementation**: Corne dongle examples in `references/example`

## Implementation Strategy

### Phase 1: Project Structure Setup

1. **Create directory structure**:

   ```
   zmk-config-leeloo_v2.1/
   ├── config/
   │   └── boards/
   │       └── shields/
   │           └── leeloo_rev2/
   │               ├── Kconfig.shield
   │               ├── Kconfig.defconfig
   │               ├── leeloo_rev2_dongle_xiao.overlay
   │               └── leeloo_rev2_dongle_xiao.conf (optional)
   ```

2. **Follow unified ZMK config template structure** as recommended in the dongle documentation

### Phase 2: Dongle Shield Definition

#### 2.1 Kconfig.shield

- Define `SHIELD_LEELOO_REV2_DONGLE_XIAO` configuration
- Follow pattern from Corne reference: `config SHIELD_CORNE_DONGLE_XIAO`

#### 2.2 Kconfig.defconfig

- Set keyboard name: "Leeloo Dongle"
- Configure as central: `ZMK_SPLIT_ROLE_CENTRAL = y`
- Set split mode: `ZMK_SPLIT = y`
- Set peripheral count: `ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS = 2` (left + right halves)
- Configure BT connections: `BT_MAX_CONN = 7` (2 peripherals + 5 BT profiles)
- Configure BT paired: `BT_MAX_PAIRED = 7`

#### 2.3 Dongle Overlay File (leeloo_rev2_dongle_xiao.overlay)

Key components needed:

- **Mock kscan**: Since dongle has no keys itself
- **Matrix transform**: Copy from `leeloo_common.dtsi` - the complete 5x12 matrix
- **Physical layout**: For ZMK Studio compatibility (future-proofing)
- **Display configuration**: Optional OLED support for dongle status

**Matrix Transform Source**: From `zmk/app/boards/shields/leeloo/leeloo_common.dtsi`:

```dts
default_transform: keymap_transform_0 {
    compatible = "zmk,matrix-transform";
    columns = <12>;
    rows = <5>;
    map = <
        RC(0,0) RC(0,1) RC(0,2) RC(0,3) RC(0,4) RC(0,5)                 RC(0,6) RC(0,7) RC(0,8) RC(0,9) RC(0,10) RC(0,11)
        RC(1,0) RC(1,1) RC(1,2) RC(1,3) RC(1,4) RC(1,5)                 RC(1,6) RC(1,7) RC(1,8) RC(1,9) RC(1,10) RC(1,11)
        RC(2,0) RC(2,1) RC(2,2) RC(2,3) RC(2,4) RC(2,5)                 RC(2,6) RC(2,7) RC(2,8) RC(2,9) RC(2,10) RC(2,11)
        RC(3,0) RC(3,1) RC(3,2) RC(3,3) RC(3,4) RC(3,5) RC(4,5) RC(4,6) RC(3,6) RC(3,7) RC(3,8) RC(3,9) RC(3,10) RC(3,11)
                                RC(4,1) RC(4,2) RC(4,3) RC(4,4) RC(4,7) RC(4,8) RC(4,9) RC(4,10)
    >;
};
```

### Phase 3: Build Configuration Updates

#### 3.1 Update build.yaml

Add dongle build target and convert existing halves to peripherals:

**Note**: `seeeduino_xiao_ble` is the **board** name for the XIAO BLE hardware, while `leeloo_rev2_dongle_xiao` is the **shield** name we'll create that defines how the Leeloo keyboard layout maps to the dongle functionality.

```yaml
include:
   # Dongle (Central)
   - board: seeeduino_xiao_ble
     shield: leeloo_rev2_dongle_xiao
     cmake-args: -DCONFIG_ZMK_KEYBOARD_NAME="Leeloo Dongle"

   # Left Half (now Peripheral)
   - board: nice_nano_v2
     shield: leeloo_rev2_left nice_view_adapter nice_view
     cmake-args: -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

   # Right Half (now Peripheral)
   - board: nice_nano_v2
     shield: leeloo_rev2_right nice_view_adapter nice_view
     cmake-args: -DCONFIG_ZMK_SPLIT=y -DCONFIG_ZMK_SPLIT_ROLE_CENTRAL=n

   # Settings Reset (required for transitioning)
   - board: nice_nano_v2
     shield: settings_reset
   - board: seeeduino_xiao_ble
     shield: settings_reset
```

### Phase 4: Display Configuration

- **Dongle Display**: Not required - dongle will function without any display
- **Keyboard Displays**: nice!view displays on both halves will continue to work normally as peripherals
- **Note**: The dongle acts purely as a wireless-to-USB bridge and doesn't need visual feedback

### Phase 5: Testing & Validation Strategy

#### 5.1 Pre-Implementation Testing

- Verify current keyboard functionality without dongle
- Document current pairing process and profiles

#### 5.2 Implementation Testing Steps

1. **Settings Reset**: Flash settings_reset to all three devices (left, right, dongle)
2. **Flash Firmware**:
   - Flash dongle firmware to XIAO BLE
   - Flash peripheral firmware to both keyboard halves
3. **Pairing Process**:
   - Reset all devices simultaneously to initiate pairing
   - Verify dongle connects to both halves
   - Test USB connection to computer
4. **Functionality Testing**:
   - Test all keys on both halves
   - Verify encoder functionality (if enabled)
   - Test nice!view displays on keyboard halves
   - Test Bluetooth profile switching on dongle

#### 5.3 Rollback Plan

- Keep backup of working firmware files
- Document process to revert to direct BT connection
- Settings reset process for "undongling"

### Phase 6: Configuration Considerations

#### 6.1 Power Management

- **Advantage**: Both keyboard halves will have improved battery life as peripherals
- **Trade-off**: Dongle must be connected for keyboard to function

#### 6.2 Latency Considerations

- Former central (left half): +1ms latency (extra USB hop)
- Right half: -6.5ms average latency (BLE to USB conversion)
- Overall: Net latency reduction for typical usage

#### 6.3 Nice!View Display Compatibility

- Maintain display functionality on both halves (displays will work normally as peripherals)
- Dongle requires no display - it functions purely as a bridge device

### Phase 7: Advanced Features (Future)

#### 7.1 ZMK Studio Integration

- Add Studio support to dongle for wireless configuration
- Include physical layout definitions for full Studio compatibility

#### 7.2 Multiple Profile Support

- Configure additional BT profiles on dongle
- Enable quick switching between host devices

## Hardware Requirements

- **Dongle Board**: XIAO BLE (seeeduino_xiao_ble) - no display needed
- **Keyboard Halves**: Current nice_nano_v2 boards with nice!view displays (unchanged)
- **Connection**: USB-C cable from dongle to computer

## Risk Assessment & Mitigation

### High Risk

- **Pairing Issues**: Mitigation - thorough settings_reset process
- **Matrix Transform Errors**: Mitigation - careful validation against working config

### Medium Risk

- **Build Process**: Mitigation - reference working examples extensively
- **Bluetooth Configuration**: Mitigation - follow established peripheral conversion patterns

### Low Risk

- **Power Management**: Well-documented behavior
- **Performance**: Proven latency improvements in most scenarios

## Success Criteria

1. ✅ Dongle successfully pairs with both keyboard halves
2. ✅ All keys function correctly through dongle
3. ✅ USB connection to computer works reliably
4. ✅ Nice!View displays remain functional on both keyboard halves
5. ✅ Bluetooth profile switching works on dongle
6. ✅ Battery life improves on keyboard halves
7. ✅ Easy rollback to original configuration possible

## Dependencies

- **Hardware**: XIAO BLE board for dongle (no display required)
- **References**: Corne dongle implementation patterns
- **ZMK Version**: Compatible with dongle feature set
- **Testing Time**: Allow for iteration on pairing and configuration

This plan provides a comprehensive roadmap for successfully implementing dongle support while maintaining all existing functionality and providing clear rollback options.
