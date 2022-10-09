# UART
## Table of contents

* [UART](#uart)
    * [Table of contents](#table-of-contents)
  * [What is it ?](#what-is-it)
  * [Identifying UART ports](#identifying-uart-ports)
    * [Using a multimeter](#using-a-multimeter)
    * [Using a logic analyzer](#using-a-logic-analyzer)
  * [Connect to serial port](#connect-to-serial-port)
    * [WARNING](#warning)
    * [Examples](#examples)
  * [UART over BLE](#uart-over-ble)

## What is it ?

UART stands for Universal asynchronous receiver transmitter. Used for serial communications over a computer or peripheral device serial port.

UART peripherals are commonly integrated in many embedded devices. UART communication makes use of baud rate to maintain synchronism between two devices. The baud rate is the rate at which information is transferred in a communication channel. 

With access to the UART, a user can see bootloader and operating system logs.

Generaly, the line is held high (at a logical 1 value) while UART is in idle state.

We call the most common configuration **8N1** : eight data bits, no parity, and 1 stop bit.

## Identifying UART ports

A UART pinout has **four** ports : 
* **TX** (Transmit)
* **RX** (Receive)
* **Vcc** (Voltage)
* **GNR** (Ground)

![](https://re-ws.pl/wp-content/uploads/2017/09/pinout.jpg)

To find UART multiple solution :
* Search on Internet
* Labeled on PCB
* Find candidates
	* Using a multimeter
	* Using a logic analyzer
* Follow PCB traces (almost always impossible)

Keep in mind that some devices **emulate** UART ports by programming the General-Purpose Input/Output (GPIO) pins if there isn't enough space on the board for dedicated hardware UART pins.

### Using a multimeter
#### GNR pin
First identify the GRN pin, by using the multimeter in continuity mode. 

Place the black probe on any grounded metallic surface, be it a part of the tested PCB or not. Then place the red probe on each of the ports. When you hear a beeping sound, you found a GND pin.

#### Vcc pin
Turn the multimeter to the DC voltage mode in and set it up to 20V of voltage. Keep the black probe on a grounded surface. Place the red probe on a suspeted pin and turn on the device.

If the multimeter measures a constant voltage of either 3.3V or 5V, you've found the Vcc pin.

#### TX pin
Keep the multimeter mode at DC voltage of 20V or less, and leave the black probe in a grounded surface. Move the red probe to the suspected pin and power cycle the device. If the voltage fluctuates for a few seconds and then stabilizes at the Vcc value, you've most likely found the TX pin.

This behavior happens because, during bootup, the device sends serial data through that TX pin for debugging purposes. Once it finishes booting, the UART line goes idle.

#### Rx pin
If you've already identified the rest of the UART pins, the nearby fourth pin is most likely the RX pin.

Otherwise, you can identify it because it has the lowest voltage fluctuation and lowest overall value of all the UART pins.

### Using a logic analyzer

!!!!!!!!!!!!!!!DO THIS PART!!!!!!!!!!

To find the UART pins we will connect the pins to a logic analyzer and look for data being transmitted. In the case of this device, bootloader and kernel logs are printed to this interface on startup, so we will expect to see data come across the transmit line of the UART device.2 This is what we will be looking for.

https://medium.com/@shubhamgolam10/reverse-engineering-uart-to-gain-shell-de9019ae427a


!!!!!!!!!!!!!!!DO THIS PART!!!!!!!!!!

## Connect to serial port
### WARNING
It's not a big deal if you confuse the UART RX and TX ports with each other, because you can easily swap the wires connecting to them without any consequences. But confusing the Vcc with the GND and connecting wires to them incorrectly **might fry the circuit**.

### Examples
![](http://remotexy.com/img/help/help-esp8266-firmware-update-usbuart.png)

![](https://vanhunteradams.com/Protocols/UART/uart_hardware.png)

!!!!!!!!!!!!!!!REDO THIS PART!!!!!!!!!!

Connect to UART using an USB to TTL, then find the `/dev/ttyUSB0` device in the `dmesg` command output. You need to create the `dialout` group for Debian or `uucp` for Manjaro :

* `sudo usermod -a -G dialout username`
* `sudo gpasswd -a username uucp`

!!!!!!!!!!!!!!!REDO THIS PART!!!!!!!!!!

#### Detect baud rate
##### Most common baud rate
The most common baud rates for UART are `9600`, `19200`, `38400`, `57600` and `115200`.

A table of other used but less common baud rates can be found here :  [Here](https://lucidar.me/en/serialib/most-used-baud-rates-table/)

##### Auto-detect the baud rate using a script
Link : [baudrate.py](https://github.com/devttys0/baudrate/blob/master/baudrate.py)
```bash
# Download
wget https://raw.githubusercontent.com/devttys0/baudrate/master/baudrate.py

# Install serial
pip2.7 install serial

# Run the script on "/dev/ttyUSB0"
python2.7 baudrate.py -p /dev/ttyUSB0
```

#### Interact with UART
Different command line tools to interract with UART :
```powershell
cu -l /dev/ttyUSB0 -s 115200
microcom -d -s 115200 -p /dev/ttyUSB0
minicom -b 115200 -o -D /dev/ttyUSB0 # To exit GNU screen, type Control-A k
screen /dev/ttyUSB0 115200
```

Script to brute force a password protected UART :
```python
import serial, time
port = "/dev/ttyUSB0"
baud = 115200
s = serial.Serial(port)
s.baudrate = baud

with open('/home/audit/Documents/IOT/passwords.lst', 'r') as f:
    lines = f.readlines()

    for pwd in lines:
        a = s.write(pwd.strip())
        print("Pwd: {}".format(pwd.strip()))
        print("Sent {} bytes".format(a))
        print("Result: {}".format(s.readline()))
        time.sleep(10)
```

## UART over BLE

It’s an emulation of serial port over BLE. The UUID of the Nordic UART Service is `6E400001-B5A3-F393-E0A9-E50E24DCCA9E`. This service exposes two characteristics: one for transmitting and one for receiving.

* **RX Characteristic** (UUID: 6E400002-B5A3-F393-E0A9-E50E24DCCA9E) : The peer can send data to the device by writing to the RX Characteristic of the service. ATT Write Request or ATT Write Command can be used. The received data is sent on the UART interface.
* **TX Characteristic** (UUID: 6E400003-B5A3-F393-E0A9-E50E24DCCA9E) : If the peer has enabled notifications for the TX Characteristic, the application can send data to the peer as notifications. The application will transmit all data received over UART as notifications.
* [nRF UART 2.0 - Nordic Semiconductor ASA](https://play.google.com/store/apps/details?id=com.nordicsemi.nrfUARTv2)
* [UART/Serial Port Emulation over BLE](https://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.sdk5.v14.0.0%2Fble_sdk_app_nus_eval.html)
* [UART Over Bluetooth Low Energy](https://thejeshgn.com/2016/10/01/uart-over-bluetooth-low-energy/)

Example with Micro::bit :

* [https://makecode.microbit.org/v1/98535-28913-33692-07418](https://makecode.microbit.org/v1/98535-28913-33692-07418)
* [Using the micro:bit Bluetooth Low Energy UART (serial over Bluetooth)](https://support.microbit.org/support/solutions/articles/19000062330-using-the-micro-bit-bluetooth-low-energy-uart-serial-over-bluetooth-)
