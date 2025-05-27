# Complete TWRP Compilation Guide - Updated 2025

*Based on Dees_Troy's original XDA guide with modern updates for Android 10-15*

## Overview

This comprehensive guide covers compiling TWRP (Team Win Recovery Project) for your Android device. TWRP is fully open source, allowing you to build custom recovery for any device with proper device tree configuration.

## Prerequisites

### System Requirements
- Ubuntu 18.04+ or similar Linux distribution
- At least 16GB RAM (32GB recommended)
- 200GB+ free disk space
- Fast internet connection

### Knowledge Requirements
- Basic Linux command line experience
- Understanding of Android build system
- Familiarity with Git and repo tool

## Build Environment Setup

### 1. Install Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y bc bison build-essential ccache curl flex \
    g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev \
    lib32readline-dev lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev \
    libsdl1.2-dev libssl-dev libwxgtk3.0-gtk3-dev libxml2 libxml2-utils \
    lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev \
    python3 python-is-python3 git-lfs aria2

# Install repo tool
sudo curl --create-dirs -L -o /usr/local/bin/repo \
    https://storage.googleapis.com/git-repo-downloads/repo
sudo chmod a+rx /usr/local/bin/repo
```

### 2. Configure Git

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### 3. Set Up Swap (Recommended)

```bash
# Create 8GB swap file
sudo dd if=/dev/zero of=/swapfile bs=1G count=8
sudo mkswap /swapfile
sudo swapon /swapfile
sudo chmod 600 /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Choose Your Build Tree

### Modern Android Versions (Android 10+)

**AOSP-based minimal manifest (Recommended for Android 10-15)**

| Android Version | Branch | Command |
|-----------------|---------|---------|
| Android 15 | twrp-15.0 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-15.0` |
| Android 14 | twrp-14.0 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-14.0` |
| Android 13 | twrp-13.0 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-13.0` |
| Android 12.1 | twrp-12.1 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1` |
| Android 11 | twrp-11 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-11` |

### Legacy Android Versions (Android 5.1-9.0)

**Omni-based manifest**

| Android Version | Branch | Command |
|-----------------|---------|---------|
| Android 9.0 | twrp-9.0 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-9.0` |
| Android 8.1 | twrp-8.1 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-8.1` |
| Android 7.1 | twrp-7.1 | `repo init -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_omni.git -b twrp-7.1` |

## Download Source Code

### 1. Initialize Repository

```bash
# Create working directory
mkdir twrp && cd twrp

# Initialize repo (example for Android 12.1)
repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-12.1

# Sync source code (this takes 2-4 hours)
repo sync -c -j$(nproc --all) --force-sync --no-clone-bundle --no-tags
```

## Device Tree Configuration

### Essential BoardConfig.mk Flags

Add these flags to your device's `BoardConfig.mk` file:

#### Core TWRP Configuration

```makefile
#######################################
# TWRP Configuration
#######################################

# Theme Selection (REQUIRED)
TW_THEME := portrait_hdpi
# Options: portrait_hdpi, portrait_mdpi, landscape_hdpi, landscape_mdpi, watch_mdpi

# Screen Resolution (for reference)
TARGET_SCREEN_WIDTH := 1080
TARGET_SCREEN_HEIGHT := 1920
```

#### Storage Configuration

```makefile
# Enable /data/media handling (most modern devices)
RECOVERY_SDCARD_ON_DATA := true

# Disable real SD card features if not present
BOARD_HAS_NO_REAL_SDCARD := true
```

#### Display and Input Fixes

```makefile
# Battery display
TW_NO_BATT_PERCENT := true              # Disable if battery % doesn't work

# Screen orientation fixes
BOARD_HAS_FLIPPED_SCREEN := true        # If screen is mounted upside-down
RECOVERY_TOUCHSCREEN_SWAP_XY := true    # Swap X/Y touch coordinates
RECOVERY_TOUCHSCREEN_FLIP_Y := true     # Flip Y axis
RECOVERY_TOUCHSCREEN_FLIP_X := true     # Flip X axis

# Custom power button mapping
TW_CUSTOM_POWER_BUTTON := 107
```

#### Boot Menu Customization

```makefile
# Remove unwanted reboot options
TW_NO_REBOOT_BOOTLOADER := true
TW_NO_REBOOT_RECOVERY := true
```

#### Advanced Features

```makefile
# Language support
TW_EXTRA_LANGUAGES := true

# Screen blanking
TW_SCREEN_BLANK_ON_BOOT := true

# Input device blacklisting (device-specific)
TW_INPUT_BLACKLIST := "hbtp_vm"

# Debugging (remove for release builds)
TWRP_EVENT_LOGGING := true

# Encryption support
TW_INCLUDE_CRYPTO := true
TW_INCLUDE_CRYPTO_FBE := true           # For File-Based Encryption (Android 7+)
TW_INCLUDE_FBE_METADATA_DECRYPT := true # For metadata decryption

# MTP support
TW_EXCLUDE_MTP := false
```

#### Modern Android Specific Flags

```makefile
# For devices with super partition (Android 10+)
BOARD_SUPER_PARTITION_SIZE := 9126805504
BOARD_SUPER_PARTITION_GROUPS := qti_dynamic_partitions

# A/B partition support
AB_OTA_UPDATER := true
BOARD_USES_RECOVERY_AS_BOOT := true
BOARD_BUILD_SYSTEM_ROOT_IMAGE := true

# Vendor boot support (Android 11+)
BOARD_INCLUDE_RECOVERY_DTBO := true
BOARD_PREBUILT_DTBOIMAGE := $(DEVICE_PATH)/prebuilts/dtbo.img
```

## Recovery.fstab Configuration

TWRP uses an fstab file to understand your device's partition layout. Modern TWRP supports both v1 and v2 fstab formats.

### Version 2 fstab (Recommended for Android 10+)

Create `/etc/recovery.fstab`:

```bash
# Android fstab file
#<src>                                    <mnt_point>    <type>  <mnt_flags and options>                              <fs_mgr_flags>
/dev/block/bootdevice/by-name/system      /system        ext4    ro,barrier=1                                         wait,slotselect,avb=vbmeta_system,logical,first_stage_mount,avb_keys=/avb/q-gsi.avbpubkey:/avb/r-gsi.avbpubkey:/avb/s-gsi.avbpubkey
/dev/block/bootdevice/by-name/vendor      /vendor        ext4    ro,barrier=1                                         wait,slotselect,avb,logical,first_stage_mount
/dev/block/bootdevice/by-name/product     /product       ext4    ro,barrier=1                                         wait,slotselect,avb,logical,first_stage_mount
/dev/block/bootdevice/by-name/userdata    /data          f2fs    noatime,nosuid,nodev,discard,inlinecrypt            wait,check,formattable,fileencryption=aes-256-xts:aes-256-cts:v2+inlinecrypt_optimized,metadata_encryption=aes-256-xts:wrappedkey_v0,keydirectory=/metadata/vold/metadata_encryption,quota,reservedsize=128M,checkpoint=fs
```

### TWRP Flags File

Create `/etc/twrp.flags` to add TWRP-specific flags:

```bash
# Boot partitions
/boot           emmc    /dev/block/bootdevice/by-name/boot                     flags=slotselect
/recovery       emmc    /dev/block/bootdevice/by-name/recovery                 flags=slotselect;backup=1
/vendor_boot    emmc    /dev/block/bootdevice/by-name/vendor_boot              flags=slotselect;backup=1;flashimg=1

# Important partitions for backup
/dtbo           emmc    /dev/block/bootdevice/by-name/dtbo                     flags=slotselect;backup=1;flashimg=1
/vbmeta         emmc    /dev/block/bootdevice/by-name/vbmeta                   flags=slotselect;backup=1;flashimg=1
/vbmeta_system  emmc    /dev/block/bootdevice/by-name/vbmeta_system           flags=slotselect;backup=1;flashimg=1

# Modem and firmware
/modem          emmc    /dev/block/bootdevice/by-name/modem                    flags=slotselect;backup=1
/bluetooth      emmc    /dev/block/bootdevice/by-name/bluetooth                flags=slotselect;backup=1
/dsp            emmc    /dev/block/bootdevice/by-name/dsp                      flags=slotselect;backup=1

# EFS and persist (critical for device identity)
/efs            ext4    /dev/block/bootdevice/by-name/efs                      flags=backup=1;display="EFS"
/persist        ext4    /dev/block/bootdevice/by-name/persist                  flags=backup=1;display="Persist"

# External storage
/external_sd    vfat    /dev/block/mmcblk1p1  /dev/block/mmcblk1               flags=storage;wipeingui;removable;display="MicroSD Card"
/usb-otg        auto    /dev/block/sda1       /dev/block/sda                   flags=storage;wipeingui;removable;display="USB-OTG"
```

### Common Fstab Flags Explained

| Flag | Description |
|------|-------------|
| `backup=1` | Include in TWRP backup/restore |
| `display="Name"` | Custom display name in TWRP |
| `storage` | Available as storage location |
| `wipeingui` | Show in advanced wipe menu |
| `removable` | Partition may not always be present |
| `slotselect` | A/B partition support |
| `flashimg=1` | Allow flashing IMG files to this partition |
| `settingsstorage` | Store TWRP settings here |

## Device Tree Structure

### Basic Device Tree Files

Your device tree should be located at `device/[manufacturer]/[codename]/` and contain:

```
device/xiaomi/spes/
├── Android.mk
├── AndroidProducts.mk
├── BoardConfig.mk
├── device.mk
├── recovery.fstab
├── recovery/
│   └── root/
│       └── system/
│           └── etc/
│               ├── recovery.fstab
│               └── twrp.flags
└── twrp_spes.mk
```

### Sample Device Tree Files

#### AndroidProducts.mk
```makefile
PRODUCT_MAKEFILES := \
    $(LOCAL_DIR)/twrp_spes.mk

COMMON_LUNCH_CHOICES := \
    twrp_spes-user \
    twrp_spes-userdebug \
    twrp_spes-eng
```

#### twrp_[codename].mk
```makefile
# Inherit from those products. Most specific first.
$(call inherit-product, $(SRC_TARGET_DIR)/product/core_64_bit.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/base.mk)

# Inherit some common TWRP stuff.
$(call inherit-product, vendor/twrp/config/common.mk)

# Inherit from device
$(call inherit-product, device/xiaomi/spes/device.mk)

PRODUCT_DEVICE := spes
PRODUCT_NAME := twrp_spes
PRODUCT_BRAND := Xiaomi
PRODUCT_MODEL := Redmi Note 11
PRODUCT_MANUFACTURER := Xiaomi

PRODUCT_GMS_CLIENTID_BASE := android-xiaomi
```

## Compilation Process

### 1. Prepare Device Tree

```bash
# Copy your device tree to the source
cp -r /path/to/your/device/tree device/[manufacturer]/[codename]/
```

### 2. Build Commands

```bash
# Set up build environment
cd twrp
export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh

# Lunch your device (replace 'spes' with your device codename)
lunch twrp_spes-eng
```

### 3. Compile TWRP

#### For devices with separate recovery partition:
```bash
mka recoveryimage
```

#### For A/B devices or devices with recovery-in-boot:
```bash
mka bootimage
```

#### For devices with vendor_boot (Android 11+):
```bash
mka vendorbootimage
```

### 4. Build Output

Your compiled recovery will be located at:
- **Recovery image**: `out/target/product/[codename]/recovery.img`
- **Boot image**: `out/target/product/[codename]/boot.img`
- **Vendor boot**: `out/target/product/[codename]/vendor_boot.img`

## Device-Specific Configurations

### Samsung Devices

Samsung devices often require special handling:

```makefile
# Samsung-specific flags
BOARD_CUSTOM_BOOTIMG_MK := device/samsung/[codename]/mkbootimg.mk
BOARD_MKBOOTIMG_ARGS := --kernel_offset 0x00008000 --ramdisk_offset 0x02000000 --tags_offset 0x01e00000

# For Samsung A/B devices
AB_OTA_UPDATER := true
BOARD_USES_RECOVERY_AS_BOOT := true

# Samsung encryption
TW_INCLUDE_CRYPTO := true
TW_INCLUDE_CRYPTO_SAMSUNG := true
```

### MediaTek Devices

```makefile
# MediaTek-specific configurations
BOARD_USES_MTK_HARDWARE := true
BOARD_HAS_MTK_HARDWARE := true

# MediaTek bootimage
BOARD_KERNEL_IMAGE_NAME := Image.gz
BOARD_INCLUDE_DTB_IN_BOOTIMG := true
BOARD_PREBUILT_DTBIMAGE_DIR := $(DEVICE_PATH)/prebuilts/dtb
```

### Qualcomm Devices

```makefile
# Qualcomm-specific flags
TARGET_USES_QCOM_BSP := true
BOARD_USES_QCOM_HARDWARE := true

# Qualcomm encryption
TARGET_HW_DISK_ENCRYPTION := true
TARGET_CRYPTFS_HW_PATH := vendor/qcom/opensource/commonsys/cryptfs_hw
```

## Troubleshooting Common Issues

### 1. Build Errors

**Error**: `lunch: command not found`
```bash
# Solution: Source the environment setup
source build/envsetup.sh
```

**Error**: `No rule to make target 'recoveryimage'`
```bash
# Solution: Your device might not have a recovery partition
# Try building bootimage instead
mka bootimage
```

### 2. Runtime Issues

**Problem**: TWRP boots but touch doesn't work
```makefile
# Add to BoardConfig.mk
RECOVERY_TOUCHSCREEN_SWAP_XY := true
RECOVERY_TOUCHSCREEN_FLIP_Y := true
```

**Problem**: Cannot decrypt /data
```makefile
# Add encryption support
TW_INCLUDE_CRYPTO := true
TW_INCLUDE_CRYPTO_FBE := true
TW_INCLUDE_FBE_METADATA_DECRYPT := true
```

**Problem**: MTP doesn't work
```makefile
# Add MTP support
TW_EXCLUDE_MTP := false
TARGET_USE_CUSTOM_LUN_FILE_PATH := /config/usb_gadget/g1/functions/mass_storage.0/lun.%d/file
```

### 3. Size Issues

If TWRP image is too large for your recovery partition:

```makefile
# Reduce image size
TW_EXCLUDE_SUPERSU := true
TW_EXCLUDE_ENCRYPTED_BACKUPS := true
BOARD_HAS_NO_REAL_SDCARD := true
TW_USE_TOOLBOX := true
```

## Testing Your Build

### 1. In Emulator (Recommended)

Always test your TWRP build in an emulator first:

```bash
# Start emulator with your recovery
emulator -avd test_device -kernel out/target/product/[codename]/kernel -ramdisk out/target/product/[codename]/ramdisk-recovery.img
```

### 2. On Real Device

**WARNING**: Always make a full backup before flashing custom recovery!

```bash
# Flash via fastboot
fastboot flash recovery recovery.img

# For A/B devices
fastboot flash boot_a boot.img
fastboot flash boot_b boot.img

# For devices with vendor_boot
fastboot flash vendor_boot_a vendor_boot.img
fastboot flash vendor_boot_b vendor_boot.img
```

## Advanced Topics

### Custom Kernel Integration

If you need a custom kernel:

1. Create `twrp.dependencies` in your device tree:
```json
[
  {
    "remote": "github",
    "repository": "username/kernel_manufacturer_codename",
    "target_path": "kernel/manufacturer/codename",
    "revision": "branch-name"
  }
]
```

2. Add to BoardConfig.mk:
```makefile
TARGET_KERNEL_SOURCE := kernel/manufacturer/codename
TARGET_KERNEL_CONFIG := codename_defconfig
TARGET_KERNEL_CLANG_COMPILE := true
```

### Building with GitHub Actions

For automated builds, you can use GitHub Actions with templates like:
- [Build-TWRP by MizProject](https://github.com/MizProject/Build-TWRP)

### Submitting to Official TWRP

To get your device officially supported:

1. Ensure your device tree compiles cleanly
2. Test thoroughly on actual hardware  
3. Submit device tree to [TeamWin GitHub](https://github.com/TeamWin/)
4. Contact TWRP maintainers with:
   - Device configuration files
   - Proof of working build
   - Testing results

## Useful Resources

### Official Sources
- [TWRP Website](https://twrp.me/)
- [TeamWin GitHub](https://github.com/TeamWin/)
- [Minimal Manifests](https://github.com/minimal-manifest-twrp)

### Community Resources
- [XDA Developers Forums](https://xdaforums.com/)
- [TWRP Building Telegram Group](https://t.me/build_twrp)
- [r/AndroidDev Reddit](https://reddit.com/r/androiddev)

### Tools
- [TWRP Device Tree Generator](https://github.com/twrpdtgen/twrpdtgen)
- [AIK (Android Image Kitchen)](https://github.com/osm0sis/Android-Image-Kitchen)

## Conclusion

Building TWRP requires patience and attention to detail, but it's a rewarding process that enables custom recovery for your device. Start with a minimal configuration and gradually add features as needed. Always test thoroughly and contribute back to the community when possible.

Remember: This is a basic guide. Each device has unique requirements that may need additional research and configuration. The Android development community is your best resource for device-specific help.

*Good luck with your TWRP build!*
