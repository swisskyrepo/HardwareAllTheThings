# Modbus

### Discovery

**Modbus Client:** 

 - [QModBus](https://sourceforge.net/projects/qmodbus/)
 - [pymodbus](https://github.com/riptideio/pymodbus)
 - [Modbus Tools](https://www.modbustools.com/) 

**Modbus Discover Nmap Script:**

```bash
nmap --script modbus-discover.nse --script-args='modbus-discover.aggressive=true' -p 502 <host>
```

**Connect to Modbus Slave:**

``` python
from pymodbus.client import ModbusTcpClient

client = ModbusTcpClient('<IP_Address_of_Target>')
client.write_coil(1, True)
result = client.read_coils(1,1)
print(result.bits[0])
client.close()

```

**Modbus Pentesting:**

 - [smod](https://github.com/0x0mar/smod)

**Modbus Slave Simulator**

 - [Diagslave](https://www.modbusdriver.com/diagslave.html) 
 - [ModbusPal](https://modbuspal.sourceforge.net/) 

**Modbus Master Simulator**

 - [modpoll](https://www.modbusdriver.com/modpoll.html)