# linux-uconsole-rpi64-arch

Arch Linux PKGBUILD for Linux kernel version 6.13 on the [ClockworkPi uConsole](https://www.clockworkpi.com/uconsole) (`aarch64`/`arm64`), based on the [Raspberry Pi Linux kernel source tree](https://github.com/raspberrypi/linux/tree/rpi-6.13.y), and [PotatoMania's work on booting Arch Linux on the uConsole](https://github.com/PotatoMania/uconsole-cm3).

This repository contains the changes needed for the `axp20x` power and `ocp8178_bl` backlight drivers to build the 6.13 kernel (based off the patches included [in PotatoMania's kernel 6.6 linux-uconsole-rpi64 package](https://github.com/PotatoMania/uconsole-cm3/tree/dev/PKGBUILDs/linux-uconsole-rpi64()).

Built for CM4, not tested on CM3 or CM4S.
