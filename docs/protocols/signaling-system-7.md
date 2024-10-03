# SS7 - Signaling System No. 7 

## Tools

* [P1sec/SigFW](https://github.com/P1sec/SigFW) - Open Source Signaling Firewall for SS7, Diameter filtering, antispoof and antisniff
* [0xc0decafe/ss7MAPer](https://github.com/0xc0decafe/ss7MAPer) - SS7 MAP (pen-)testing toolkit
* [SigPloiter/SigPloit](https://github.com/SigPloiter/SigPloit) - SigPloit: Telecom Signaling Exploitation Framework - SS7, GTP, Diameter & SIP


## SMS 2FA Interception

SS7 plays a part in the transportation of SMS messages. An attacker may be able to register a victims `MSISDN` (mobile number) on a fake `MSC` (Mobile Switching Centre), the victims operator's `HLR` (Home Location Register) that works as a kind of telephone directory for `MSISDNs`, operators and SMS service centres (`SMSC`) will set the new location for the Victim’s `MSISDN`.

When, for this example the victims Bank sends them a 2FA authentication token the MSC transfers the SMS to the `SMSC` the real `MSMSC` asks the victims operator's `HLR` for the victims location, the `HLR` replies with the attacker operated `MSC`. The real operator's `SMSC` transfers the SMS to the fake `MSC` operated by the attack.


## SMS Spoofing

One of the simplest and most accessible attacks is SMS spoofing, which doesn't require direct access to the SS7 network. Many people are unaware that the "from" field in an SMS message lacks authentication, allowing it to be easily forged. The sender can insert any alphanumeric word into the "from" section of a message.

SMS spoofing attacks can be carried out with minimal cost by using an SMS gateway service, many of which are accessible on the clear web. According to SOS Intelligence, most of these services lack abuse monitoring or prevention mechanisms. As a result, it’s possible to send spoofed messages to a victim—much like phishing emails—prompting them to take action, often at little to no cost.


## Location Tracking

Within the SS7 network of a network operator it may be possible to request the `LAC` (Location Area Code) and `Cell ID` and with that information get a reasonably good location for a victim. However, this may require the prior knowledge of the subscribers `IMEI` (International Equipment Identity) or/and `IMSI` (International Mobile Subscriber Identity) – A `MSISDN` alone may not be sufficient to be able to query this information.


## References

* [Exposing The Flaw In Our Phone System - Veritasium - 22 sept. 2024](https://youtu.be/wVyu7NB7W6Y)
* [SS7 VULNERABILITIES AND ATTACK EXPOSURE REPORT - 2018](https://www.gsma.com/get-involved/gsma-membership/wp-content/uploads/2018/07/SS7_Vulnerability_2017_A4.ENG_.0003.03.pdf)
* [A Step by Step Guide to SS7 Attacks - Adam Weinberg - April 30, 2023](https://www.firstpoint-mg.com/blog/ss7-attack-guide/)
* [An investigation into SS7 Exploitation Services on the Dark Web - Amir Hadzipasic - November 17, 2021](https://sosintel.co.uk/an-investigation-into-ss7-exploitation-services-on-the-dark-web/)
* [SS7 ATTACK - Ahmet Göker - Apr 28, 2022](https://shadowintel.medium.com/ss7-attack-a068f45ef83f)
* [SCTPscan - Finding entry points to SS7 Networks & Telecommunication Backbones - Philippe Langlois - 19 Apr 2007](https://www.blackhat.com/presentations/bh-europe-07/Langlois/Presentation/bh-eu-07-langlois-ppt-apr19.pdf)
* [ss7MAPer – A SS7 pen testing toolkit - Daniel Mende - February 16, 2016](https://insinuator.net/2016/02/ss7maper-a-ss7-pen-testing-toolkit/)