# HyberusFD_HUB
Hyberus=Hydra+Cerberus
HyberusFD_HUB is an open-source USB-to-CAN hub based on the STM32G0B1CBT6. It supports a 12 V–24 V power input, provides three USB 2.0 ports via the CH334R chip, and offers two CAN connections through TJA1051K/3 chips. It is commonly used with MSM8916-based 4G USB modems as a USB expansion and CAN interface extension. The firmware supports both CAN and CAN FD protocols.

- Pinout

|feature|pin|
|------|------|
|LED_RX|PA0|
|LEX_TX|PA1|
|USB_P|PA12|
|USB_N|PA11|
|CAN1_RX|PB9|
|CAN1_TX|PB8|
|CAN1_S|PC13|

![image](https://github.com/qpzm1258/candleLightFD_fw/blob/HyberusFD_HUB/hardware/HyberusFD_HUB/2D_HyberusFD_HUB.png)

![image](https://github.com/qpzm1258/candleLightFD_fw/blob/HyberusFD_HUB/hardware/HyberusFD_HUB/3D_HyberusFD_HUB.png)

# candleLight_gsusb
[![Build](https://github.com/qpzm1258/candleLightFD_fw/actions/workflows/ci.yml/badge.svg)](https://github.com/qpzm1258/candleLightFD_fw/actions)
This is firmware for HyberusFD_HUB base on [candleLight](https://github.com/marckleinebudde/candleLight_fw/tree/multichannel)

## Building

Building requires arm-none-eabi-gcc toolchain.

```shell
sudo apt-get install gcc-arm-none-eabi

mkdir build
cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=../cmake/gcc-arm-none-eabi-8-2019-q3-update.cmake

# or,
# cmake-gui ..
# don't forget to specify the cmake toolchain file before configuring.
#
# compile all targets :

make

# OR, each board target is a cmake option and can be disabled before running 'make';
# OR, compile a single target , e.g.
make cantact_fw

#
# to list possible targets :
make help

```

## Download Binaries
Prebuilt binaries can be downloaded by clicking [![CI](https://github.com/candle-usb/candleLight_fw/actions/workflows/ci.yml/badge.svg)](https://github.com/candle-usb/candleLight_fw/actions). On the workflow overview page, select the latest workflow that ran on master branch. The firmware artifacts can downloaded by clicking them at the bottom of the page.

## Flashing

Flashing candleLight on linux: (source: [https://cantact.io/cantact/users-guide.html](https://cantact.io/cantact/users-guide.html))
- Flashing requires the dfu-util tool. On Ubuntu, this can be installed with `sudo apt install dfu-util`.
- compile as above, or download the current binary release: gsusb_cantact_8b2b2b4.bin
- If dfu-util fails due to permission issues on Linux, you may need additional udev rules. Consult your distro's documentation and see `70-candle-usb.rules` provided here.

### recommended simple method
- If compiling with cmake, `make flash-<targetname_fw>`, e.g. `make flash-canable_fw`, to invoke dfu-util.

### method for reflashing a specific device by serial
- when multiple devices are connected, dfu-util may be unable to choose which one to flash.
- Obtain device's serial # by looking at `dfu-util -l`
- adapt the following command accordingly :
 `dfu-util -D CORRECT_FIRMWARE.bin -S "serial_number_here", -a 0 -s 0x08000000:leave`
- note, the `:leave` suffix above may not be supported by older builds of dfu-util and is simply a convenient way to reboot into the normal firmware.

### fail-safe method (or if flashing a blank device)
- Disconnect the USB connector from the CANtact, short the BOOT pins, then reconnect the USB connector. The device should enumerate as "STM32 BOOTLOADER".

- invoke dfu-util manually with: `sudo dfu-util --dfuse-address -d 0483:df11 -c 1 -i 0 -a 0 -s 0x08000000 -D CORRECT_FIRMWARE.bin` where CORRECT_FIRMWARE is the name of the desired .bin.
- Disconnect the USB connector, un-short the BOOT pins, and reconnect.



## Associating persistent device names
With udev on linux, it is possible to assign a device name to a certain serial number (see udev manpages and [systemd.link](https://www.freedesktop.org/software/systemd/man/systemd.link.html)).
This can be useful when multiple devices are connected at the same time.

An example configuration :

```
 $ cat /etc/systemd/network/60-persistent-candev.link
[Match]
Property=ID_MODEL=cannette_gs_usb ID_SERIAL_SHORT="003800254250431420363230"

[Link]
# from systemd.link manpage:
# Note that specifying a name that the kernel might use for another interface (for example "eth0") is dangerous because the name assignment done by udev will race with the assignment done by the kernel, and only one
#   interface may use the name. Depending on the order of operations, either udev or the kernel will win, making the naming unpredictable. It is best to use some different prefix

Name=cannette99
```

( The serial number can be found with the `lsusb` utility). After reloading systemd units and resetting this board :

```
 $ ip a
....
59: cannette99: <NOARP,ECHO> mtu 16 qdisc noop state DOWN group default qlen 10
    link/can
 $
```


## Hacking
### Submitting pull requests
- Each commit must not contain unrelated changes (e.g. functional and whitespace changes)
- Project must be compilable (with default options) and functional, at each commit.
- Squash any "WIP" or other temporary commits.
- Make sure your editor is not messing up whitespace or line-ends.
- We include both a `.editorconfig` and `uncrustify.cfg` which should help with whitespace.

Typical command to run uncrustify on all source files (ignoring HAL and third-party libs):
`uncrustify -c ./uncrustify.cfg --replace --no-backup $(find include src -name "*.[ch]")`

### Profiling
Not great on cortex-M0 cores (F042, F072 targets etc) since they lack hardware support (ITM and SWO). However, it's possible to randomly sample the program counter and get some coarse profiling info.

For example, openocd has the `profile` command (see https://openocd.org/doc/html/General-Commands.html#Misc-Commands), e.g.

```profile 5 test.out 0x8000000 0x8100000```

(from inside gdb, the command needs to be prefixed with `monitor` to forward it to openocd, i.e. `monitor profile 5 .....`.

The .out file can then be processed with `gprof <firmware_name> -l test.out`



## Links to related projects
* [Cangaroo](https://github.com/HubertD/cangaroo) open source can bus analyzer software
* [Candle.NET](https://github.com/elliotwoods/Candle.NET) .NET wrapper for the candle API
