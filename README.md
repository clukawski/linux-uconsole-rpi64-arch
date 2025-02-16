# linux-uconsole-rpi64-arch

The contents of this repository is a WIP, use at your own risk. See [known issues](#known-issues) below.

Arch Linux PKGBUILD for Linux kernel version 6.13 on the [ClockworkPi uConsole](https://www.clockworkpi.com/uconsole) (`aarch64`/`arm64`), based on the [Raspberry Pi Linux kernel source tree](https://github.com/raspberrypi/linux/tree/rpi-6.13.y), and [PotatoMania's work on booting Arch Linux on the uConsole](https://github.com/PotatoMania/uconsole-cm3).

This repository contains the changes needed for the `axp20x` power and `ocp8178_bl` backlight drivers to build the 6.13 kernel (based off the patches included [in PotatoMania's kernel 6.6 linux-uconsole-rpi64 package](https://github.com/PotatoMania/uconsole-cm3/tree/dev/PKGBUILDs/linux-uconsole-rpi64)).

Built and testing on a CM4 uConsole, do not have CM3 or CM4S hardware for validation.

## Known Issues

- DSI panel not properly initialized
  - ```
    [    8.971301] panel-clockwork-cwu50 fe700000.dsi.0: error -EBUSY: Failed to request GPIO (-16)
    [    8.971325] panel-clockwork-cwu50 fe700000.dsi.0: probe with driver panel-clockwork-cwu50 failed with error -16
	  ```
- Potential issue with the asp20x driver + patches
  - ```
    [   21.474449] platform axp20x-battery-power-supply.2.auto: deferred probe pending: (reason unknown)
    ```

## How to Build

This assumes you're building this on an `x86_64` version of Arch Linux. If you're building this on an `arm64` Arch Linux host, running `makepkg` should suffice.

### Dependencies

```
pacman -S aarch64-linux-gnu-gcc aarch64-linux-gnu-binutils
```

### Build Package

```
makepkg CARCH=aarch64 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```
