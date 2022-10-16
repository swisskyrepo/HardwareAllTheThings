# MQTT

### Discovery

MQTT client:

* mqtt-spy
* [MQTT CLI](https://asciinema.org/a/DlPmJwXbhuAURHseamGdMy4z3/embed?speed=2\&autoplay=true)
* [MQTT Lens](https://chrome.google.com/webstore/detail/mqttlens/hemojaaeigabkbcookmlgmdigohjobjm)
* MQTT.fx
* mosquitto_tools

Scan an MQTT with nmap : `nmap -p 1883 -vvv --script=mqtt-subscribe -d sensors.domain.com`

```powershell
mosquitto_sub -h sensors.domain.com -t '#'
mosquitto_sub -h sensors.domain.com -t '+'
mosquitto_sub -h sensors.domain.com -t "/sensor/"
```

### Explore MQTT

Connect and subscribe to every topics using the `#` keyword.

```python
import paho.mqtt.client as mqtt
def on_connect(client, userdata, flags, rc):
   print "[+] Connection successful"
   client.subscribe('#', qos = 1)        # Subscribe to all topics
   client.subscribe('$SYS/#')            # Broker Status (Mosquitto)
def on_message(client, userdata, msg):
   print '[+] Topic: %s - Message: %s' % (msg.topic, msg.payload)
client = mqtt.Client(client_id = "MqttClient")
client.on_connect = on_connect
client.on_message = on_message
client.connect('SERVER IP HERE', 1883, 60)
client.loop_forever()
```

Send MQTT requests

```python
import paho.mqtt.client as mqtt
def on_connect(client, userdata, flags, rc):
   print "[+] Connection success"
client = mqtt.Client(client_id = "MqttClient")
client.on_connect = on_connect
client.connect('IP SERVER HERE', 1883, 60)
client.publish('smarthouse/garage/door', "{'open':'true'}")
```

### MQTT Fuzzing 

* [MQTT-Fuzz](https://github.com/F-Secure/mqtt_fuzz)