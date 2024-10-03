# ESP32

![ESP32](../assets/esp32-pinout.png)


* [ESP32 datasheet: esp32_datasheet_en.pdf](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
* [Xtensa®Instruction Set Architecture (ISA)](https://0x04.net/~mwk/doc/xtensa.pdf)

ESP32 and ESP8266 share almost the same architecture.


## Tools

* [espressif/esptool](https://github.com/espressif/esptool) - Espressif SoC serial bootloader utility
* [jmswrnr/esp32knife](https://github.com/jmswrnr/esp32knife) - Tools for ESP32 firmware dissection
* [scientifichackers/ampy](https://github.com/scientifichackers/ampy) - Utility to interact with a MicroPython board over a serial connection
* [ESPWebTool](https://esp.huhn.me/) - Flash your ESP32 or ESP8266 through your browser.
* [tenable/esp32_image_parser](https://github.com/tenable/esp32_image_parser) - A toolkit for helping you reverse engineer ESP32 firmware.


## Firmwares

* [risinek/esp32-wifi-penetration-tool](https://github.com/risinek/esp32-wifi-penetration-tool) - Exploring possibilities of ESP32 platform to attack on nearby Wi-Fi networks. 
* [justcallmekoko/ESP32Marauder](https://github.com/justcallmekoko/ESP32Marauder) - A suite of WiFi/Bluetooth offensive and defensive tools for the ESP32 


## Flashing

The ESP32 microprocessor uses the Xtensa instruction set, use `Tensilica Xtensa 32-bit little-endian` in Ghidra.

* Flash a new firmware with `espressif/esptool`
    ```ps1
    esptool.py -p /dev/ttyUSB0 -b 460800 --before default_reset --after hard_reset --chip esp32  write_flash --flash_mode dio --flash_size 2MB --flash_freq 40m 0x1000 build/bootloader/bootloader.bin 0x8000 build/partition_table/partition-table.bin 0x10000 build/ble_ctf.bin
    esptool.py -p /dev/ttyS5 -b 115200 --after hard_reset write_flash --flash_mode dio --flash_freq 40m --flash_size detect 0x8000 build/partition_table/partition-table.bin 0x1000 build/bootloader/bootloader.bin 0x10000 build/esp32-wifi-penetration-tool.bin
    ```

* Flash a new firmware with `scientifichackers/ampy` (MicroPython)
    ```ps1
    ampy --port /dev/ttyUSB0 put bla.py
    ```

* Dump the flash
    ```ps1
    esptool -p COM7 -b 115200 read_flash 0 0x400000 flash.bin
    ```

* Dissect the flash
    ```ps1
    python esp32knife.py --chip=esp32 load_from_file ./flash.bin
    ```

* Flash the new firmware
    ```ps1
    # repair the checksum
    python esp32fix.py --chip=esp32 app_image ./patched.part.3.factory 
    esptool -p COM7 -b 115200 write_flash 0x10000 ./patched.part.3.factory.fixed
    ```


## References

* [ESP32-reversing - BlackVS](https://github.com/BlackVS/ESP32-reversing)
* [ESP32 Wi-Fi Penetration Tool - GitHub - Exploring possibilities of ESP32 platform to attack on nearby Wi-Fi networks](https://github.com/risinek/esp32-wifi-penetration-tool)
* [ESP32 Wi-Fi Penetration Tool - Documentation - Exploring possibilities of ESP32 platform to attack on nearby Wi-Fi networks](https://risinek.github.io/esp32-wifi-penetration-tool/)
* [Hacking a Smart Home Device - @jmswrnr - 03 Feb 2024](https://jmswrnr.com/blog/hacking-a-smart-home-device)
* [Reversing ESP8266 Firmware (Part 1) - Bored Pentester - 26th October 2018](https://boredpentester.com/reversing-esp8266-firmware-part-1/)
* [Reversing ESP8266 Firmware (Part 2) - Bored Pentester - 25th October 2018](https://boredpentester.com/reversing-esp8266-firmware-part-2/)
* [Reversing ESP8266 Firmware (Part 3) - Bored Pentester - 25th October 2018](https://boredpentester.com/reversing-esp8266-firmware-part-3/)
* [Reversing ESP8266 Firmware (Part 4) - Bored Pentester - 25th October 2018](https://boredpentester.com/reversing-esp8266-firmware-part-4/)
* [Reversing ESP8266 Firmware (Part 5) - Bored Pentester - 25th October 2018](https://boredpentester.com/reversing-esp8266-firmware-part-5/)
* [Reversing ESP8266 Firmware (Part 6) - Bored Pentester - 25th October 2018](https://boredpentester.com/reversing-esp8266-firmware-part-6/)