# JTAG

### Summary

* JTAG Pins
* JTAGEnum
* References

### JTAG Pins

> Allows testing, debugging, firmware manipulation and boundary scanning

**TCK: Test Clock** The drummer, or metronome that dictates the speed of the TAP controller. Voltage on this pin simply pulses up and down in a rhythmic, steady beat. On every “beat” of the clock, the TAP controller takes a single action. The actual clock speed is not specified in the JTAG standard. The TAP controller accepts its speed from the outside device controlling JTAG.

**TMS: Test Mode Select** Voltages on the Mode Select pin control what action JTAG takes. By manipulating the voltage on this pin, you tell JTAG what you want it to do.

**TDI: Test Data-In** The pin that feeds data into the chip. The JTAG standard does not define protocols for communication over this pin. That is left up to the manufacturer. As far as JTAG is concerned, this pin is simply an ingress method for 1s and 0s to get into the chip. What the chip does with them is irrelevant to JTAG.

**TDO: Test Data-Out** The pin for data coming out of the chip. Like the Data-In pin, communication protocols are not defined by JTAG. TRST: Test Reset (Optional) This optional signal is used to reset JTAG to a known good state, we'll explain why this is optional in a few paragraphs.

AVR has lock bits that protects device from extracting flash

* Removing this lockbits will erase entire device
* If you have them set, you’re not lucky, try to get firmware from other sources

```powershell
# Read fuses and lock bits using avarice –r
$ avarice --program --file test.elf --part atmega128 --jtag /dev/ttyUSB0 :4444
# Acquire firmware using avrdude
$ avrdude -p m128 -c jtagmkI –P /dev/ttyUSB0 -U flash:r:”/home/avr/flash.bin":r
```

### JTAGEnum

JTAGenum is an open source Arduino JTAGenum.ino or RaspbberyPi JTAGenum.sh (experimental) scanner. This code was built with three primary goals:

* Given a large set of pins on a device determine which are JTAG lines
* Enumerate the Instruction Register to find undocumented functionality

⚠️ JTAG and device must share the same ground.

Software Connection Set up:

* Download the INO sketch from the github
* Open the Arduino IDE and Load the downloaded JTAGEnum sketch
* Choose the correct Serial Port and Board
* Compile and Upload the sketch
* Open the Serial Monitor
* Set the correct baud rate
* Enter the command to scan ("s")

Arduino PIN Layout

* Digital PIN 2(Black)
* Digital PIN 3(White)
* Digital PIN 4(Grey)
* Digital PIN 5(Maroon)
* Digital PIN 6(Blue)
* GND - GREEN

![](https://3.bp.blogspot.com/-OmjCNFWbnf0/WKx4NEjfb9I/AAAAAAAADy8/-qz5Of4iDbcT5mtonl6st1hVGrmsGUs4gCLcB/s640/FOUND.png)

### References

* [JTAGulator vs. JTAGenum, Tools for Identifying JTAG Pins in IoT Devices by Dylan Ayrey](https://www.praetorian.com/blog/jtagulator-vs-jtagenum-tools-for-identifying-jtag-pins-in-iot-devices?edition=2019)
* [JTAG PIN Identification - February 21, 2017](https://just2secure.blogspot.com/2017/02/jtag-pin-identification.html)
* [Hardware Debugging for Reverse Engineers Part 2: JTAG, SSDs and Firmware Extraction - Posted Apr 2, 2020 by wrongbaud](https://wrongbaud.github.io/posts/jtag-hdd/)
