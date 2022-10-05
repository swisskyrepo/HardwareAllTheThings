# UART

### Summary

* What is it ?
* Identifying UART Ports
* Connect to serial port
  * Detect baudrate
  * Interact with the /dev/ttyUSB0
* UART over BLE
* Examples

## What is it ?
UART stands for Universal asynchronous receiver transmitter. Used for serial communications over a computer or peripheral device serial port.

UART peripherals are commonly integrated in many embedded devices. UART communication makes use of baud rate to maintain synchronism between two devices. The baud rate is the rate at which information is transferred in a communication channel. 

With access to the UART, a user can see bootloader and operating system logs.

Generaly, the line is held high (at a logical 1 value) while UART is in idle state.

We call the most common configuration 8N1 : eight data bits, no parity, and 1 stop bit.

## Identifying UART Ports

A UART pinout has four ports : 
- TX (Transmit)
- Rx (Receive)
- Vcc (Voltage)
- GNR (Ground)

To find UART multiple solution :
- Search on Internet
- Labeled on PCB
- Find candidates
	- Use a multi-meter
- Follow PCB traces (almost always impossible)

Keep in mind that some devices emulate UART ports by programming the Generla-Purpose Input/Output (GPIO) pins if there isn't enough space on the board for dedicated hardware UART pins.

### Use a multimeter

#### GNR pin
First identify the GRN pin, by using the multimeter in continuity mode. 

Place the black probe on any grounded metallic surface, be it a part of the tested PCB or not. Then place the red probe on each of the ports. When you hear a beeping sound, you found a GND pin.

#### VCC pin
Turn the multimeter to the DC voltage mode in and set it up to 20V of voltage. Keep the black probe on a grounded surface. Place the red probe on a suspeted pin and turn on the device.

If the multimeter measures a constant voltage of either 3.3V or 5V, you've found the VCC pin.

#### Tx pin
Keep the multimeter mode at DC voltage of 20V or less, and leave the black probe in a grounded surface. Move the red probe to the suspected pin and power cycle the device. If the voltage fluctuates for a few seconds and then stabilizes at the Vcc value, you've most likely found the Tx pin.

This behavior happens because, during bootup, the device sends serial data through that Tx pin for debugging purposes. Once it finishes booting, the UART line goes idle.

#### Rx pin
If you've already identified the rest of the UART pins, the nearby fourth pin is most likely the Rx pin.

Otherwise, you can identify it because it has the lowest voltage fluctuation and lowest overall value of all the UART pins.

### Connect to serial port

Connect to UART using an USB to TTL, then find the `/dev/ttyUSB0` device in the `dmesg` command output. You need to create the `dialout` group for Debian or `uucp` for Manjaro :

* `sudo usermod -a -G dialout username`
* `sudo gpasswd -a username uucp`

#### Detect baudrate

Standard baud rate are `110`, `300`, `600`, `1200`, `2400`, `4800`, `9600`, `14400`, `19200`, `38400`, `57600`, `115200`, `128000` and `256000`.\
Auto-detect baud rate using the script : [devttys0/baudrate/baudrate.py](https://github.com/devttys0/baudrate/blob/master/baudrate.py)

#### Interact with the /dev/ttyUSB0

```powershell
cu -l /dev/ttyUSB0 -s 9600
screen port_name 115200
minicom -b 115200 -o -D Port_Name # to exit GNU screen, type Control-A k.
microcom -d -s 9600 -p /dev/ttyUSB0
microcom -d -s 19200 -p /dev/ttyUSB0
```

```python
import serial, time
port = "/dev/ttyUSB0"
baud = 9600
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

### UART over BLE

It’s an emulation of serial port over BLE. The UUID of the Nordic UART Service is `6E400001-B5A3-F393-E0A9-E50E24DCCA9E`. This service exposes two characteristics: one for transmitting and one for receiving.

* **RX Characteristic** (UUID: 6E400002-B5A3-F393-E0A9-E50E24DCCA9E) : The peer can send data to the device by writing to the RX Characteristic of the service. ATT Write Request or ATT Write Command can be used. The received data is sent on the UART interface.
* **TX Characteristic** (UUID: 6E400003-B5A3-F393-E0A9-E50E24DCCA9E) : If the peer has enabled notifications for the TX Characteristic, the application can send data to the peer as notifications. The application will transmit all data received over UART as notifications.
* [nRF UART 2.0 - Nordic Semiconductor ASA](https://play.google.com/store/apps/details?id=com.nordicsemi.nrfUARTv2)
* [UART/Serial Port Emulation over BLE](https://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.sdk5.v14.0.0%2Fble_sdk_app_nus_eval.html)
* [UART Over Bluetooth Low Energy](https://thejeshgn.com/2016/10/01/uart-over-bluetooth-low-energy/)

Example with Micro::bit :

* [https://makecode.microbit.org/v1/98535-28913-33692-07418](https://makecode.microbit.org/v1/98535-28913-33692-07418)
* [Using the micro:bit Bluetooth Low Energy UART (serial over Bluetooth)](https://support.microbit.org/support/solutions/articles/19000062330-using-the-micro-bit-bluetooth-low-energy-uart-serial-over-bluetooth-)

### Examples

![](https://developer.android.com/things/images/raspberrypi-console.png) 
![](http://remotexy.com/img/help/help-esp8266-firmware-update-usbuart.png)
