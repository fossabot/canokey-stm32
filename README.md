# Canokey on STM32
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fcanokeys%2Fcanokey-stm32.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fcanokeys%2Fcanokey-stm32?ref=badge_shield)


Canokey is an open-source USB security token, providing the following functions:

- OpenPGP Card V3.4 (RSA, ECDSA, ED25519 supported)
- PIV Card
- TOTP / HOTP (RFC6238 / RFC4226)
- U2F
- FIDO2 (WebAuthn)

It works on modern Linux/Windows/macOS operating systems without additional driver required.

## Hardware

Current implementation is based on STM32L423KC MCU, which features a Cortex-M4 processor, 256KB Flash, 64 KB SRAM, and a full-speed USB controller. For demonstration purposes, you may run this project on the [NUCLEO-L432KC](https://os.mbed.com/platforms/ST-Nucleo-L432KC/) development board with the following hardware connection:

- D2(PA12) <-> USB D+
- D10(PA11) <-> USB D-
- GND <-> USB GND
- 5V <-> USB Power

The microUSB port on board is for ST-LINK, do not confuse it with USB signal pins.

## Build the Firmware

Prerequisites:

- CMake >= 3.6
- GNU ARM Embedded Toolchain, downloaded from [ARM](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads)
- git (used to generate embedding version string)

Build steps:

```shell
# clone this repo and all the submodules
# in the top-level folder
mkdir build
cd build
cmake -DCROSS_COMPILE=<path-to-toolchain>/bin/arm-none-eabi- \
    -DCMAKE_TOOLCHAIN_FILE=../toolchain.cmake \
    -DCMAKE_BUILD_TYPE=Release ..
make canokey.bin
```

Then download the firmware file `canokey.bin` to the STM32 with flash programming tools.

## Initialize and Test

Prerequisites:

- Linux OS with pcscd, pcsc_scan and scriptor installed

Connect the Canokey to PC, an USB CCID device should show up. The `pcsc_scan` command should be able to detect a smart card:

```
$ pcsc_scan
TODO: output
```

Then, initialize the Canokey by running `./init-fido-demo.sh`. This script will set admin PIN to `123456`, set serial number to current timestamp, and write an attestation certificate used by FIDO. Refer to [admin doc](https://canokeys.github.io/doc/development/protocols/admin/) if you want to customize these values.

```
$ ./init-fido-demo.sh
TODO: output
```

After initialization, you are free to use Canokey with applications, such as:

- GPG, e.g. `gpg --card-status`
- SSH pkcs11
- piv-tool
- Websites with U2F enabled, e.g. https://github.com/settings/security
- Websites with WebAuthn enabled, e.g. https://webauthn.me/
- TOTP dashboard (under development)

## Internals

The hardware-independent code (including applets, storage, cryptography and USB stack) is in a submodule named [canokey-core](https://github.com/canokeys/canokey-core).

Major hardware-dependent components in this repo:

- `Drivers`: STM32 HAL Drivers
- `Src/device.c`: Hardware operations called by canokey-core
- `Src/lfs_init.c`: Flash operations used by file system
- `Src/main.c`: Hardware initialization
- `Src/usbd_conf.c`: USB interface


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fcanokeys%2Fcanokey-stm32.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fcanokeys%2Fcanokey-stm32?ref=badge_large)