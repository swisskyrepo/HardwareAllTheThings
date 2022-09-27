---
cover: >-
  https://images.unsplash.com/photo-1526304640581-d334cdbbf45e?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=2970&q=80
coverY: 0
---

# UART

****

### Summary

* UART
* Connect to serial port
  * Detect baudrate
  * Interact with the /dev/ttyUSB0
* UART over BLE
* Examples

### UART

> Investigations began by attaching a multimeter to the outputs and reading the voltage. Three of the outputs read 3.3V and the fourth 0.02V. The fourth pin was ground. A UART or serial interface typically consists of 4 pins, power, transmit, receive and ground it was therefore hypothesized that the pins maybe utilised for this purpose.

```powershell
# sudo pip3 install pyserial
VCC <-> 5V
RX <-> TX
TX <-> RX
GND <-> GND
```

The ground can be found out using the continuity test in multimeter. Usually all the GROUND PINs of every component/chip of a device are connected to each other and are also connected with almost all the metal parts of the hardware. We can easily understand which are the GROUND PINs by connecting every PIN with a metal part and verifying if the current flows. The header that produces beep sound on being touched is the ground (ohmmeter).

The VCC can be found out using the voltage test. One of the pin is kept on the identified ground and other on any of the 3 pins. If you get a constant high voltage means that it is a VCC pin .

### Connect to serial port

Connect to UART using an USB to TTL, then find the `/dev/ttyUSB0` device in the `dmesg` command output. You need to create the `dialout` group for Debian or `uucp` for Manjaro :

* `sudo usermod -a -G dialout username`
* `sudo gpasswd -a username uucp`

#### Detect baudrate

Standard baud rate are `110`, `300`, `600`, `1200`, `2400`, `4800`, `9600`, `14400`, `19200`, `38400`, `57600`, `115200`, `128000` and `256000`.\
Auto-detect baud rate using the script : https://github.com/devttys0/baudrate/blob/master/baudrate.py

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
* nRF UART 2.0 - Nordic Semiconductor ASA - https://play.google.com/store/apps/details?id=com.nordicsemi.nrfUARTv2
* Specifications - https://infocenter.nordicsemi.com/index.jsp?topic=%2Fcom.nordic.infocenter.sdk5.v14.0.0%2Fble\_sdk\_app\_nus\_eval.html
* https://thejeshgn.com/2016/10/01/uart-over-bluetooth-low-energy/

Example with Micro::bit :

* https://makecode.microbit.org/v1/98535-28913-33692-07418
* https://support.microbit.org/support/solutions/articles/19000062330-using-the-micro-bit-bluetooth-low-energy-uart-serial-over-bluetooth-

### Examples

![](https://developer.android.com/things/images/raspberrypi-console.png) ![](http://remotexy.com/img/help/help-esp8266-firmware-update-usbuart.png)
