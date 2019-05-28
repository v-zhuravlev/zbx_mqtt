# MQTT Zabbix integration

This is MQTT <-> Zabbix integration tested with Wirenboard controllers. This solution uses dependent items, jsonpath and jsonpath validation.  
Collects multiple MQTT topics at once. Topic wildcards can be used as well. So single call to the device/broker can be to collect everything you need.  
Zabbix 4.2 is recommended ( where jsonpath validation is used to find out out-of-date values) and at least 3.4 for basic usage without validation.

## Setup with Wirenboard

### Zabbix server/proxy (externalscripts)

`apt-get install python python-pip`
`pip install paho-mqtt==1.2.3 --no-cache-dir`
`cp mqtt /usr/lib/zabbix/externalscripts/`
`chmod +x /usr/lib/zabbix/externalscripts/mqtt`

### Zabbix agent (userparameteres)

NOT SUPPORTED. it's currently problematic to install paho-mqtt on wirenboard. So skip and try zabbix server/proxy setup.  
First option is to install script on wireboard itself
on wirenboard:
Note: If Debian Wheezy, remove all lines in `/etc/apt/sources.list` and add:
`deb http://archive.debian.org/debian/ wheezy main`
`apt-get update`
`apt-get install python python-pip`
`pip install paho-mqtt==1.2.3 --no-cache-dir`

### Test by example

In order to get WB-MS sensor data using wirenboard

run:

`mqtt -t '/devices/wb-ms-thls_25/#' --mqtt-host={HOST.CONN} -s 1`
`mqtt -t '/devices/wb-ms-thls_25/#' --mqtt-host=<MQTT_BROKER_IP> -s 1`

If sucessful, you will get the output like this:

```json
{
	"/devices/wb-ms-thls_25/meta/name": "WB-MS-THLS 25",
	"/devices/wb-ms-thls_25/controls/Temperature": "27.7",
	"/devices/wb-ms-thls_25/controls/Temperature/meta/type": "temperature",
	"/devices/wb-ms-thls_25/controls/Temperature/meta/readonly": "1",
	"/devices/wb-ms-thls_25/controls/Temperature/meta/order": "1",
	"/devices/wb-ms-thls_25/controls/Humidity": "26.9",
	"/devices/wb-ms-thls_25/controls/Humidity/meta/type": "rel_humidity",
	"/devices/wb-ms-thls_25/controls/Humidity/meta/readonly": "1",
	"/devices/wb-ms-thls_25/controls/Humidity/meta/order": "2",
	"/devices/wb-ms-thls_25/controls/Illuminance": "0",
	"/devices/wb-ms-thls_25/controls/Illuminance/meta/type": "lux",
	"/devices/wb-ms-thls_25/controls/Illuminance/meta/readonly": "1",
	"/devices/wb-ms-thls_25/controls/Illuminance/meta/order": "3",
	"/devices/wb-ms-thls_25/controls/Sound Level": "51.12",
	"/devices/wb-ms-thls_25/controls/Sound Level/meta/type": "sound_level",
	"/devices/wb-ms-thls_25/controls/Sound Level/meta/readonly": "1",
	"/devices/wb-ms-thls_25/controls/Sound Level/meta/order": "4",
	"/devices/wb-ms-thls_25/controls/Input Voltage": "18.072",
	"/devices/wb-ms-thls_25/controls/Input Voltage/meta/type": "voltage",
	"/devices/wb-ms-thls_25/controls/Input Voltage/meta/readonly": "1",
	"/devices/wb-ms-thls_25/controls/Input Voltage/meta/order": "5"
}
```

Congratulations! You have just got a lot of MQTT topics with single call and it us wrapped in JSON now. You can proceed to building Zabbix template.

### Zabbix template


