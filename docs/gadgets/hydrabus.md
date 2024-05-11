# HydraBus

![HydraBUS - Pin Assignment](../assets/hydrabus_pin_assignment.jpg)


## Features

Support many extensions:

- [HydraNFC](https://swisskyrepo.github.io/HardwareAllTheThings/gadgets/hydranfc/) - Hydrabus shield for NFC
- [HydraFlash](https://swisskyrepo.github.io/HardwareAllTheThings/gadgets/hydraflash/) - NAND flash shield
- [HydraLINCAN](https://github.com/smillier/HydraLINCAN) - Hydrabus shield for LIN and CAN buses

External interactions:

- [Bus interaction commands](https://github.com/hydrabus/hydrafw/wiki/Bus-interaction-commands)
- [Trigger mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-Trigger-guide)
- [ADC guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-ADC-guide)
- [DAC guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-DAC-guide)
- [PWM guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-PWM-guide)
- [GPIO guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-GPIO-guide)
- [SPI guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-SPI-guide) / [binary mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-Binary-SPI-mode-guide)
- [I2C guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-I2C-guide) / [binary mode]()
- [1-wire guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-1-wire-guide) / [binary mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-binary-1-Wire-mode-guide)
- [2-wire guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-2wire-guide)
- [3-wire guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-3wire-guide)
- [UART guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-UART-guide) / [binary mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-binary-UART-mode-guide)
- [CAN guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-CAN-guide) / [binary mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-Binary-CAN-mode-guide)
- [JTAG guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-JTAG-guide)
- [NAND Flash guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-NAND-Flash-guide) / [binary mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-binary-NAND-Flash-mode-guide)
- [Wiegand guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-Wiegand-guide)
- [LIN guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-LIN-guide)
- [SMARTCARD guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-SMARTCARD-guide) / [binary mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-binary-SMARTCARD-mode-guide)
- [NFC guide](https://github.com/hydrabus/hydrafw/wiki/HydraFW-HydraNFC-v1-guide) / [binary mode](https://github.com/hydrabus/hydrafw/wiki/HydraFW-binary-NFC-Reader-mode-guide)


## Firmware

* [hydrabus/hydrafw](https://github.com/hydrabus/hydrafw) - HydraFW official firmware for HydraBus/HydraNFC
* [hydrabus/hydrafw_hydranfc_shield_v2](https://github.com/hydrabus/hydrafw_hydranfc_shield_v2) - HydraFW dedicated to HydraBus v1 / HydraNFC Shield v2
* [bvernoux/blackmagic](https://github.com/bvernoux/blackmagic) - In application debugger for ARM Cortex microcontrollers

### Firmware Update

Detailed steps: [hydrafw/Getting-Started-with-HydraBus-flash-and-use-hydrafw-on-linux](https://github.com/hydrabus/hydrafw/wiki/Getting-Started-with-HydraBus-flash-and-use-hydrafw-on-linux)

1. Install `dfu-util`
    ```ps1
    git clone git://git.code.sf.net/p/dfu-util/dfu-util dfu-util
    cd dfu-util
    ./autogen.sh
    ./configure
    sudo make install
    ```

2. Download the latest release of the firmware
    ```ps1
    wget https://github.com/hydrabus/hydrafw/releases/download/v0.11/build_HydraFW_v0.11-12-ga6019f4_HydraBus_HydraNFC.zip
    wget https://raw.githubusercontent.com/hydrabus/hydrafw/master/utils/udev-rules/09-hydrabus.rules -O ~/hydrafw/09-hydrabus.rules
    ```

3. Keep pressing `UBTN` button at `PowerON/RESET` in order to enter `USB DFU`
4. Connect the MicroUSB cable from your PC to HydraBus
5. Check Linux detection for HydraBus in DFU mode: `sudo dfu-util -l`
6. Flash the firmware: `sudo dfu-util -i 0 -a 0 -d 0483:df11 -D ./build/hydrafw.dfu`


## Commands

* Basic info: `show system`

    ```ps1> show system
    HydraFW (HydraBus) v0.11-1-g4d74500 2023-05-09
    sysTime: 0x000d82dd.
    cyclecounter: 0x76ac02b9 cycles.
    cyclecounter64: 0x0000000076ac02cb cycles.
    10ms delay: 1680035 cycles.
    ```

* Determine the port name: `ls -l /dev/tty*`
* Interact witht the HydraBus: `screen /dev/ttyACM0`
* Switch to SPI mode: `spi`
* Determine the pin for SPI: `show pins`


## Syntax

| Value | Description |
|-------|-------------|
| [	    | Chip select (CS) active (low) |
| ]	    | CS disable (high) |
| r	    | Read one byte by sending dummy byte (0xff). r:1...255 for bulk reads |
| hd    | Read one byte by sending dummy byte (0xff). hd:1...4294967295 for bulk reads. Displays a hexdump of the result |
| w	    | Followed by values to write byte(s). w:1...255 for bulk writes |
| 0b    | Write this binary value. Format is 0b00000000 for a byte, but partial bytes are also fine: 0b1001 |
| 0	    | Write this Octal value. Format is prefixed by a 0 (values from 000 to 077) |
| "     | Write an ASCII-encoded string |
| 0h/0x | Write this HEX value. Format is 0h01 or 0x01. Partial bytes are fine: 0xA. A-F can be lower-case or capital letters |
| 0-255	| Write this decimal value. Any number not preceded by 0x, 0h, or 0b is interpreted as a decimal value |

Examples:

* Read Identification (0x9F): `[ 0x9F r:3 ]`
* Read Data (0x03) at the address (0x00:3) and read 32 bytes (hd:32) `[ 0x03 0x00:3 hd:32 ]`


## References

* [HydraBus/HydraFW wiki - Benjamin Vernoux - Jan 21, 2021](https://github.com/hydrabus/hydrafw/wiki/)
* [HydraBus v1.0 Specifications - HydraBus](https://hydrabus.com/hydrabus-1-0-specifications)
* [HydraBus Assembly Video - Lab401 - 30 may 2017](https://youtu.be/9lFEPG8EG6w)
* [BlackAlps17: Hydrabus: Lowering the entry fee to the IoT bugfest - Benjamin Vernoux -  2 dec. 2017](https://www.youtube.com/watch?v=theYbzPhYH8)
* [HydraBus - An Open Source Platform - RMLL Sec 2017](https://archives.pass-the-salt.org/RMLL%20Security%20Tracks/2017/slides/RMLL-Sec-2017-hydrabus.pdf)
* [Ph0wn, my first IoT CTF - Part 3 - Sebastien Andrivet - Dec. 19, 2018](https://sebastien.andrivet.com/en/posts/ph0wn-my-first-iot-ctf-part-3/)
* [Getting Started with HydraBus flash and use hydrafw on linux - Benjamin Vernoux - 05/02/2024](https://github.com/hydrabus/hydrafw/wiki/Getting-Started-with-HydraBus-flash-and-use-hydrafw-on-linux)