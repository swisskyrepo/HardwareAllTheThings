# MMS (IEC 61850)

### Discovery

***MMS Client***

 - [Lib 61850](https://github.com/mz-automation/libiec61850)
 - [IEC 61850 Open Server](https://github.com/robidev/iec61850_open_server)

***MMS Discovery Nmap Script***

Source: [mms-identify.nse](https://github.com/atimorin/scada-tools/blob/master/mms-identify.nse)

```bash
nmap -d --script mms-identify.nse --script-args='mms-identify.timeout=500' -p 102 <target_host>
```

### Explore MMS 

- [MMS Client Example](https://libiec61850.com/documentation/iec-61850-client-tutorial/)
- [MMS Server Example](https://libiec61850.com/documentation/iec-61850-server-tutorial/)

### Fuzzing MMS 

- [61850-fuzzing](https://github.com/fkie-cad/61850-fuzzing) 
