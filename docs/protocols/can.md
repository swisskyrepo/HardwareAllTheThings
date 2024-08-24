# CAN Bus

The Controller Area Network (CAN) bus is a high-integrity serial communication protocol designed for real-time data exchange in embedded systems, particularly in automotive and industrial applications. It operates on a multi-master, message-oriented architecture, allowing multiple devices (nodes) to communicate on the same network without a central controller.


## Interact

```ps1
pip install python-can
pip install python-can-utils
```

```py
import can
bus = can.Bus()
while True:
    msg = can.Message(3, data=[0 for _ in range(8)])
    bus.send(msg)
```

* [Tbruno25/can-explorer](https://github.com/Tbruno25/can-explorer) - Visualize CAN Bus messages in real time 


## UDS

> Unified Diagnostic Services (UDS) is a communication protocol used in automotive Electronic Control Units (ECUs) to enable diagnostics, firmware updates, routine testing and more.

### Implementation

![UDS message structure](../assets/uds-message-frame-can-bus.svg)

* [pylessard/python-udsoncan](https://github.com/pylessard/python-udsoncan) - Python implementation of UDS (ISO-14229) standard.
* [driftregion/iso14229](https://github.com/driftregion/iso14229) - C implementation of ISO 14229 (UDS) server and client for embedded systems


### SID

| UDS SID (Request)	| UDS SID (Response)	|  UDS Service	| Details | 
| ----------------- | --------------------- | ------------- | ------- |  
| 0x10	| 0x50	| Diagnostic session control | Control which UDS services are available. | 
| 0x11	| 0x51	| ECU Reset | It resets the ECU (includes hard reset, key off and soft reset) | 
| 0x27	| 0x67	| Security access | It enables use of security critical services via authentication. | 
| 0x28	| 0x68	| Communication control | This field turns send/receive of messages ON or OFF in the ECU. | 
| 0x29	| 0x69	| Aunthentication | Enables more advanced authentication vs. 0x27 (PKI based exchange). | 
| 0x3E	| 0x7E	| Tester present | Send a heartbeat message periodically to remain in existing session . | 
| 0x83	| 0xC3	| Access timing parameters | View/Modify timing parameters used in client/server communication. | 
| 0x84	| 0xC4	| Secured Data Transmission | Send encrypted data via ISO 15764 (extended data link security) | 
| 0x85	| 0xC5	| Control DTC Settings | Enable/Disable detection of errors (e.g. used during diagnostics) | 
| 0x86	| 0xC6	| Response On Event | Request that ECU processes a service request if an event happens | 
| 0x87	| 0xC7	| Link Control | Set the baud rate for diagnostic access |
| 0x22	| 0x62	| Read Data by Identifier | Read data from targetted ECU - e.g. VIN, sensor data etc. |
| 0x23	| 0x63	| Read Data by Address | Read data from physical memory (e.g. to understand software behaviour) |
| 0x24	| 0x64	| Read Scaling Data By Identifier | Read information about how to scale data identifiers |
| 0x2A	| 0x6A	| Read Data by Identifier Periodic | Request ECU to broadcast sensor data at slow/medium/fast/stop rate |
| 0x2C	| 0x6C	| Dynamically Define Data Identifier | Define data parameter for use in 0x22 or 0x2A dynamically |
| 0x2E	| 0x6E	| Write Data By Identifier | Program specific variables determined by data parameters |
| 0x3D	| 0x7D	| Write Memory By address | Write information to the ECU's memory |
| 0x14	| 0x54	| Clear Diagnostic Information | Delete stored DTCs |
| 0x19	| 0x59	| Read DTC Information | Read stored DTCs as well as related information |
| 0x2F	| 0x6F	| Input Output Control By Identifier | Gain control over ECU analog/digital inputs/outputs |
| 0x31	| 0x71	| Routine Control | Initiate/stop routines (e.g. self testing, erasing of flash memory) |
| 0x34	| 0x74	| Request Download | Start request to add software/data to ECU (including location/size) |
| 0x35	| 0x75	| Request Upload | Start request to read software/data from ECU (including location/size) |
| 0x36	| 0x76	| Transfer Data | Perform actual transfer of data following use of 0x74/0x75 |
| 0x37	| 0x77	| Request Transfer Exit | Stop the transfer of data |
| 0x38	| 0x78	| Request File Transfer | Perform a file download/upload to/from the ECU |
| 0x7F	| Negative Response | Send with a negative response code when a request can not be handled. |


## References

* [Awesome CAN bus tools, hardware and resources - iDoka](https://github.com/iDoka/awesome-canbus)
* [UDS SID Table | UDS SID Request And Response - rfwireless-world](https://www.rfwireless-world.com/Terminology/UDS-SID-Table.html)
* [UDS Explained - A Simple Intro (Unified Diagnostic Services) - csselectronics](https://www.csselectronics.com/pages/uds-protocol-tutorial-unified-diagnostic-services)
* [Unified Diagnostic Services (UDS) Explained - A Simple Intro [2022] - csselectronics](https://youtu.be/CV_B8tJgI5E)