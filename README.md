# illumos SolARM

Inofficial fork of https://github.com/Toasterson/illumos-aarch64-devenv adding wip and artifacts that are not part of the upstream development atm.

The dlc directory contains a pre-build image (SD-Card 16GB) for those in giving illumos ARM a testdrive while not having the possibility to setup 
the development environment.

Remember that the current illumos port only supports an UART console, Gbe and MMC. No USB (keyboard/mouse), no video (console framebuffer, framebuffer,
accelerated video), no audio, no power management, etc. You have to connect to the UART GPIO's of your Pi4 (GND, TXD, RXD) and will be able to access via
for favorite terminal to the Pi and the illumos console (e.g. screen /dev/ttyUSB0 115200).

hostname: illumos-rpi
login: root/rpi

The pre-build image runs on Pi4 hardware (Pi 4B, Pi400, CM4)

# Howto use

Extract illumos-rpi.img.xz and dd to an empty Micro-SD.

Start your termnal on your favorite system and connect to the Pi UART.

Insert the Micro-SD into your Pi and power on.

U-Boot will start interrupt boot process by pressing a key (else it will try to do an enet boot).

In the U-Boot command prompt enter: run mmc_boot

It should start booting from the secondary partiton on the Micro-SD card (zfs) and after a few seconds you should be welcomed with the login prompt.

root/rpi will grant you access and now enjoy exploring illumos SolARM.

# We need your support

Your background does not matter, so many areas where support is more than welcome. We currently need every helping hand that feels comfortable in
something that still has to evolve.

# References

https://www.illumos.org/books/dev/

https://dlc.openindiana.org/docs/osol/20090715/DRIVER/html/docinfo.html

Thx

