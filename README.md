# MQTT Zabbix integration

This is MQTT <-> Zabbix integration, tested with Wirenboard controllers. This solution uses dependent items, JSONPath preprocessing and JSONPath validation.  
Collects multiple MQTT topics at once using topic wildcards like `#`. So single subscribe to the device/MQTT broker can be used to collect everything you need.  
Zabbix 4.2 is recommended ( where JSONPath validation is used to ignore out out-of-date values) and at least 3.4 for the basic usage without validation.

For Zabbix 4.4+ see also this https://github.com/v-zhuravlev/zbx_plugin_mqtt

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

In order to get data from [WB-MS sensor](https://wirenboard.com/en/product/WB-MS/) data using wirenboard

run:

`./mqtt -t '/devices/wb-ms-thls_25/#' --mqtt-host=<MQTT_BROKER_IP> -s 1`

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

Congratulations! You have just got all of the topics available on WB-MS sensor with single MQTT subscribe and data is wrapped in JSON now. In such form, it would be very easy to retrieve data in Zabbix using JSONPath. Let's proceed to building Zabbix template.

### Zabbix template

First, create master item. for example for WB-MS-THLS sensor:

![image](https://user-images.githubusercontent.com/14870891/58502621-8e58c700-818f-11e9-8223-370aaa5f46b4.png)
This item should have:  
Type=External check  
Key=`mqtt["-t=<topic with wildcard>","--mqtt-host=<ip address of mqtt broker>"]`  
Type of information=Text  
History storage period=1d

Then, proceed with creating dependent items for each sensor value you are interested. For example, humidity:

![image](https://user-images.githubusercontent.com/14870891/58510183-e1874580-81a0-11e9-9fe1-32b8091e4fec.png)
Type=Dependent item  
Master item=`<WB-MS-THLS get>`  - item from previous step  
Key=doesn't really matter  
And in preprocessing tab, apply two steps:  
![image](https://user-images.githubusercontent.com/14870891/58510832-3d060300-81a2-11e9-97a5-eab5097a39d1.png)
1. In wirenboard, you can check for value errors. For example if controller failed to collect requested value in time. For that use 'Check for error in JSON' step. It checks for `<value>/meta/error` topic. If this topic is present then there were problems collecting `<value>` and so further processing in zabbix is stopped. Value present on controller is ignored since it is unknown how long ago it was actually collected by wirenboard.  
2. If there is no `<value>/meta/error` we can now use JSONPath preprocessing to get the actual `<value>` from MQTT topic.  

Now create dependent items for other sensor data: temperature, sound, light. You will get something like this:  
![image](https://user-images.githubusercontent.com/14870891/58502009-4be2ba80-818e-11e9-92de-524971b25c76.png)
As you can see, all timestamps are synchronized as data is received simultaneously with single MQTT request.

## References

https://www.zabbix.com/documentation/current/manual/config/items/itemtypes/dependent_items  
https://www.zabbix.com/documentation/current/manual/config/items/item#item_value_preprocessing  
https://blog.zabbix.com/zabbix-3-4-mass-data-collection-using-mercury-and-smartmontools-as-an-example/5784/  
