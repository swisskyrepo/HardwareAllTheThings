# Bluetooth

### Challenge

* BLE HackMe (https://www.microsoft.com/store/apps/9N7PNVS9J1B7) - works with nRF Connect (Android), [Introduction\_to\_BLE\_security](http://smartlockpicking.com/slides/HITB\_Cyberweek\_2020\_A\_Practical\_Introduction\_to\_BLE\_security.pdf) / https://github.com/smartlockpicking/BLE\_HackMe

### Bluetooth configuration for Kali Linux

```powershell
$ sudo apt-get install bluetooth blueman bluez
$ sudo systemctl start bluetooth
$ sudo hciconfig hci0 up

$ sudo hcitool lescan
00:1A:7D:DA:71:06 Ph0wn Beacon
25:55:84:20:73:70 (unknown
```

Apt doesn't have a recent version of bluez, recompile it with the following lines.

```powershell
wget https://www.kernel.org/pub/linux/bluetooth/bluez-5.18.tar.xz
dpkg --get-selections | grep -v deinstall | grep bluez
tar xvf bluez-5.18.tar.xz
sudo apt-get install libglib2.0-dev libdbus-1-dev libusb-dev libudev-dev libical-dev systemd libreadline-dev
.configure --enable-library
make -j8 && sudo make install
sudo cp attrib/gatttool /usr/local/bin/
```

### Enumerate services and characteristics

> BLE is based on specification called General Attribute profile (GATT), that defines how communication/data transfer between client and server.

```powershell
sudo apt-get install git build-essential libglib2.0-dev python-setuptools
git clone https://github.com/IanHarvey/bluepy.git
cd bluepy
python setup.py build
sudo python setup.py install
git clone git clone https://github.com/hackgnar/bleah
cd bleah
python setup.py build
sudo python setup.py install

sudo bleah -b $MAC -e
```

Using bettercap

```powershell
sudo bettercap -eval "net.recon off; events.stream off; ble.recon on"
ble.show
ble.enum 04:52:de:ad:be:ef
```

Using expliot

```powershell
# List of Services
run ble.generic.scan -a <mac address> -s
# List of characteristics
run ble.generic.scan -a <mac address> -c
```

Using gatttool, we can enumerate the services and their characteristics, use `sudo gatttool -b $MAC -I` to have an interactive gatttool shell:

* Services: They are set of provided features and associated behaviors to interact with the peripheral. Each service contains a collection of characteristics.
* Characteristics: Characteristics are defined attribute types that contain a single logical value

```powershell
MAC=30:AE:A4:2A:54:8A

$ gatttool -b $MAC --primary
attr handle = 0x0001, end grp handle = 0x0005 uuid: 00001801-0000-1000-8000-00805f9b34fb
attr handle = 0x0014, end grp handle = 0x001c uuid: 00001800-0000-1000-8000-00805f9b34fb
attr handle = 0x0028, end grp handle = 0xffff uuid: 000000ff-0000-1000-8000-00805f9b34fb
# Services whose UUID start with 00001801 and 00001800 are special values defined in the norm
# The other is a custom one which holds the CTF

$ gatttool -b $MAC --characteristics
handle = 0x0002, char properties = 0x20, char value handle = 0x0003, uuid = 00002a05-0000-1000-8000-00805f9b34fb
handle = 0x0015, char properties = 0x02, char value handle = 0x0016, uuid = 00002a00-0000-1000-8000-00805f9b34fb
```

### Read BLE data

Read data with gatttool

```powershell
$ sudo gatttool -b $MAC -I
[00:1A:7D:DA:71:06][LE]> connect

# list characteristics
[00:1A:7D:DA:71:06][LE]> characteristics
handle: 0x000b, char properties: 0x0a, char value handle: 0x000c, uuid: 4b796c6f-5265-6e49-7342-61644a656469

# read characteristic at char handle
[00:1A:7D:DA:71:06][LE]> char-read-hnd 0x000c
Characteristic value/descriptor: 44 65 63 72 79 70 74 20 74 68 65 20 6d 65 73 73 61 67 65 2c 20 77 72 69 74 65 20 74 68 65 20 64 65 63 72 79 70 74 65 64 20 76 61 6c 75 65 20 61 6e 64 20 72 65 61 64 20 62 61 63 6b 20 74 68 65 20 72 65 73 70 6f 6e 73 65 20 74 6f 20 66 6c 61 67 2e 20 45 6e 63 72 79 70 74 65 64 20 6d 65 73 73 61 67 65 3a 20 63 34 64 33 32 38 36 35 37 61 39 64 62 33 64 66 65 39 31 64 33 36 36 36 62 39 34 31 62 33 36 31

# one liner
$ gatttool -b $MAC --char-read -a 0x002a|awk -F':' '{print $2}'|tr -d ' '|xxd -r -p;printf '\n'
```

### Read BLE notification/indication

```powershell
$ gatttool -b $MAC -a 0x0040 --char-write-req --value=0100 --listen
$ gatttool -b $MAC -a 0x0044 --char-write-req --value=0200 --listen
```

### Write BLE data

Write data with bettercap

```powershell
ble.recon on
ble.write 04:52:de:ad:be:ef 234bfbd5e3b34536a3fe723620d4b78d ffffffffffffffff
```

Write data with gatttool

```powershell
$ gatttool -b $MAC --char-write-req -a 0x002c -n $(echo -n "12345678901234567890"|xxd -ps)

# With char-write, we perform a Write Command and don't expect a response from the server
# With char-write-req, we perform a Write Request and expect a response from the server
$ gatttool -b $MAC -a 0x0050 --char-write-req --value=$(echo -n 'hello' | xxd -p)

# inside gatttool shell
[00:1A:7D:DA:71:06][LE]> char-write-req 0x000c 476f6f64205061646177616e21212121
[00:1A:7D:DA:71:06][LE]> char-read-hnd 0x000c
Characteristic value/descriptor: 43 6f 6e [...] 2e
```

### Change Bluetooth MAC

```powershell
$ bdaddr -r 11:22:33:44:55:66
$ gatttool -I -b E8:77:6D:8B:09:96 -t random
```

### Sniff Bluetooth communication

#### Using Ubertooth

:warning: You need 3 ubertooth.

```powershell
ubertooth-btle -U 0 -A 37 -f  -c bulb_37.pcap
ubertooth-btle -U 1 -A 38 -f  -c bulb_38.pcap
ubertooth-btle -U 2 -A 39 -f  -c bulb_39.pcap
```

#### Using Micro::Bit

* https://media.defcon.org/DEF%20CON%2025/DEF%20CON%2025%20presentations/DEF%20CON%2025%20-%20Damien-Cauquil-Weaponizing-the-BBC-MicroBit.pdf

#### Using Android HCI

Enable the Bluetooth HCI log on the device via Developer Options—also from the SDK, there is a helpful tool called the **Bluetooth HCI snoop log** (available after version 4.4)

> It works like a hook in the stack to capture all the HCI packets in a file. For most Android devices, the log file is at /sdcard/btsnoop\_hci.log or /sdcard/oem\_log/btsnoop/

```powershell
$ adb pull /sdcard/oem_log/btsnoop/<your log file>.log
```
