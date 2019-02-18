---
title: "Week 9 Roundup"
date: 2017-03-14
layout: post
---

I've spent the last week setting up (fighting with) the GCC compiler and
FreeRTOS SDK for the Wemos D1 mini development board (powered by the ESP8266
chipset).

I first tried to install the toolchain myself, which involves compiling a
modified version of the GCC/binutils toolchain (`xtensa-lx106-elf-gcc`). This
failed rather quickly -- building the toolchain requires a case-sensitive
filesystem, which I don't have (macOS 10.12 uses HFS+), and is still apparently
complicated with a non-gcc host compiler (I use clang/LLVM).

The workaround was to go through the Arduino IDE to obtain the toolchain's
binaries. After adding the ESP8266 repository to the IDE's library manager
(`http://arduino.esp8266.com/stable/package_esp8266com_index.json`) and
installing the Arduino/ESP8266 support package, I just copied the compiler
files to my own toolchains folder (`~/toolchains/esp8266`). I also installed
`esptool.py`, a more stable replacement for the upload/flash tool that comes
with the toolchain.

Installing the FreeRTOS SDK was harder. I was able to compile and upload some
example projects from the [Espressif-supplied SDK][1], but that was about it.
Compiling anything relies on black-hole shell scripts and tens of recursive
Makefiles. I could have tried to work with that, but I'd rather use a build
system that I can at least partially understand, should anything fail.

The solution was to use [ESP Open RTOS][2], a community-driven FreeRTOS SDK for
the ESP. The build system is still more complicated than I'd like, and it's
nigh-impossible to compile with a Makefile built from scratch, but importing
the SDK's `common.mk` does the trick (and I can understand most of that
Makefile, which means I should be able to debug problems that will inevitably
arise).

 [1]: https://github.com/espressif/ESP8266_RTOS_SDK
 [2]: https://github.com/pfalcon/esp-open-sdk
