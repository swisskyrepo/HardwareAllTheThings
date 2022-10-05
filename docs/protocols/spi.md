# SPI

SPI is a serial peripheral interface. The controller selects a chip it send and receive information to and from. NOR flash chips with an SPI interface are commonly used as firmware boot chip. SPI has one read and one write line. In QSPI mode, 4 lines are used in parallel.

### Dump Firmware via SPI

```powershell
sudo raspi-confi > Interface > SPI(P4)
NOTE: might need a press/hold the reset button
# check
sudo flashrom -p linux spi:dev=/dev/spidev0.0,spispeed=1000
# dump
sudo flashrom -p linux spi:dev=/dev/spidev0.0,spispeed=1000 -r dump.bin
```

An ESP8266 and ESP32 have several SPI busses available in hardware, SPI0 is hooked up to it's own internal flash and is not intended for use, but the HSPI and VSPI busses can be used in combination with a SOIC-8 clamp to read from SPI NOR chips. cheap clips have a tendency to jump off the chips, pomona 5250 has a better grip.

```powershell
$ python ./esptool.py read_flash --spi-connection HSPI 0 0x400000 flash_dump.bin
```

### SPIFFS

```powershell
$ cd ~/.arduino15/packages/esp32/tools/esptool/2.3.1
$ python ./esptool.py -p /dev/ttyUSB0 -b 460800 read_flash 0x300000 0x0fb000 /tmp/spiffs.bin

$ cd ~/.arduino15/packages/esp32/tools/mkspiffs/0.2.3
$ ./mkspiffs -u /tmp/data -p 256 -b 8192 -s 1028096 /tmp/spiffs/bin
```

### ESP32 Diagrams

Color coded which pins can be connected from the ESP HSPI pins to an SPI flash. The pink interfaces (DQ1 and DQ2) are optional, they are only used in QSPI mode.

<p align="center">
  <img src="https://raw.githubusercontent.com/swisskyrepo/HardwareAllTheThings/master/docs/assets/Esp32.png" style="max-width: 400px;"><br />
  <img src="https://raw.githubusercontent.com/swisskyrepo/HardwareAllTheThings/master/docs/assets/Qspi.png" style="max-width: 400px;">
</p>

### References

* https://www.youtube.com/watch?v=Bn5zajZ4I5E
