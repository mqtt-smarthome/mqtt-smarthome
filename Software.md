# mqtt-smarthome

_Note that this project is not associated with or endorsed by http://mqtt.org_

Home: https://github.com/mqtt-smarthome

## List of Software written with this proposal in mind

If you create a mqtt interface that follows the mqtt-smarthome architecture proposal it would be nice if you add this badge to your readme: [![mqtt-smarthome](https://img.shields.io/badge/mqtt-smarthome-blue.svg)](https://github.com/mqtt-smarthome/mqtt-smarthome)

```
[![mqtt-smarthome](https://img.shields.io/badge/mqtt-smarthome-blue.svg)](https://github.com/mqtt-smarthome/mqtt-smarthome)
```

### Interfaces

* [airtunes2mqtt](https://github.com/hobbyquaker/airtunes2mqtt) - MQTT controlled Multi-Room Audio with Airplay/Airtunes Devices.
* [almond2mqtt](https://github.com/nathanielc/almond2mqtt) - Control your home automation devices connected to your Almond+ router from Securifi.
* [bcontrol2mqtt](https://github.com/hobbyquaker/bcontrol2mqtt) - Publish values from [TQ Energy Manager](http://www.tq-group.com/produkte/produktdetail/prod/energy-manager/extb/Main/) to MQTT.
* [bravia2mqtt](https://github.com/forty2/bravia2mqtt) - Control your Sony Bravia TV with MQTT.
* [buderus2mqtt](https://github.com/krambox/buderus2mqtt) - Bridge between a KM200 Buderus internet gateway and MQTT with state an write access.
* [chromoflex2mqtt](https://github.com/owagner/chromoflex2mqtt) - Interface for Barthelme Chromoflex RGB LED controllers
* [cul2mqtt](https://github.com/hobbyquaker/cul2mqtt) - Interface between [Busware CUL](http://shop.busware.de/product_info.php/cPath/1/products_id/29) (FS20, HMS, EM, ...) and MQTT.
* [dashbutton2mqtt](https://github.com/hobbyquaker/dashbutton2mqtt) - Publish dash button presses to MQTT.
* [eno2mqtt](https://github.com/owagner/eno2mqtt) - Interface between an Enocean USB300 (TCM310) adapter and MQTT.
* [evohome2mqtt](https://github.com/svrooij/evohome2mqtt) - MQTT Interface for the Honeywell Evohome system.
* [flowerpower2mqtt](https://github.com/hobbyquaker/flowerpower2mqtt) - Connect  [Parrot Flower Power](http://www.parrot.com/usa/products/flower-power/) plant sensors to MQTT
* [haiku2mqtt](https://github.com/forty2/haiku2mqtt) - A bridge between Haiku smart fans and MQTT.
* [helios2mqtt](https://github.com/mreschka/helios2mqtt) - A deamon for syncing a helios easy controls system like my KWL EC 220D to mqtt.
* [hm2mqtt](https://github.com/owagner/hm2mqtt) - Interface between EQ-3's Homematic line of smarthome devices and MQTT.
* [hm2mqtt.js](https://github.com/hobbyquaker/hm2mqtt.js) - Interface between EQ-3's Homematic line of smarthome devices and MQTT. Also Supports Homematic IP and ReGa Programs/Variables.
* [homekit2mqtt](https://github.com/hobbyquaker/homekit2mqtt) - HAP-NodeJS based MQTT-HomeKit Bridge. Control devices with Siri and HomeKit enabled Apps.
* [homely](https://github.com/baol/homely) - Early alpha stage collection of Go bridges and wiring for MQTT. Telegram and Desktop notification, Geofencing, Simple wiring logic, Interface with Arduino, integration with Domoticz (rerouting Domoticz mqtt messages to more descriptive topics)
* [hue2mqtt](https://github.com/owagner/hue2mqtt) - Interface between the Philips Hue bridge and MQTT.
* [hue2mqtt](https://github.com/svrooij/hue2mqtt) alternative - Interface between the Philips Hue bridge and MQTT, build in Node.js by svrooij.
* [hue2mqtt.js](https://github.com/hobbyquaker/hue2mqtt.js) alternative - Interface between the Philips Hue bridge and MQTT, build in Node.js by hobbyquaker.
* [HS100toMQTT](https://github.com/dersimn/HS100toMQTT) - Gateway between TPLink HS100/HS110 and MQTT.
* [ipcam2mqtt](https://github.com/svrooij/ipcam2mqtt) - Use your IP-Cameras as motion/sound sensors with this bridge.
* [knx2mqtt](https://github.com/owagner/knx2mqtt) - Interface between the KNX home automation standard and MQTT. Uses the Calimero KNX library.
* [kobold2mqtt](https://github.com/krambox/kobold2mqtt) - Bridge between a Vorwerk VR200 internet gateway and MQTT with state an write access.
* [kodi2mqtt](https://github.com/owagner/kodi2mqtt) - Interface between a Kodi mediacenter instance and MQTT.
* [lgtv2mqtt](https://github.com/hobbyquaker/lgtv2mqtt) - Interface between LG WebOS Smart TVs and MQTT.
* [loxone2mqtt](https://github.com/krambox/loxone2mqtt) - Bridge between a Loxone miniserver and MQTT.
* [lirc2mqtt](https://github.com/hobbyquaker/lirc2mqtt) - Send and Receive Infrared Commands via MQTT and [LIRC](http://www.lirc.org).
* [miflora-mqtt-daemon](https://github.com/thomdietrich/miflora-mqtt-daemon) - Publish Xiaomi Mi Flora sensor data to MQTT.
* [modbus2mqtt](https://github.com/owagner/modbus2mqtt) - Modbus master which publishes register values via MQTT.
* [mqtt2atlonamatrix](https://github.com/forty2/mqtt2atlonamatrix) - Control Atlona HDMI matrix switches with MQTT
* [mqtt2elasticsearch](https://github.com/hobbyquaker/mqtt2elasticsearch) - Send MQTT messages to Elasticsearch.
* [mqtt2homekit](https://github.com/forty2/mqtt2homekit) - Roughly the opposite of [homekit2mqtt](https://github.com/hobbyquaker/homekit2mqtt): Control your HomeKit-enabled devices with MQTT and without Siri or iPhone.
* [mqtt2tivoremote](https://github.com/forty2/mqtt2tivoremote) - Make TiVo DVR remote control available through an mqtt-smarthome style interface.
* [mqtt-dmx-sequencer](https://github.com/hobbyquaker/mqtt-dmx-sequencer) - Control DMX devices via Art-Net by MQTT   
* [onkyo2mqtt](https://github.com/owagner/onkyo2mqtt) - Interface between Onkyo AVR's EISCP network remote protocol and MQTT. Uses the onkyo-eiscp library.
* [owrtwifi2mqtt](https://github.com/dersimn/owrtwifi2mqtt) - Using your OpenWRT Router's Wifi to detect if a person's smartphone is still in/near the apartment and publish via MQTT
* [rpi2mqtt](https://github.com/hobbyquaker/rpi2mqtt) - Connect a RaspberryPis GPIOs and 1-Wire Temperature Sensors to MQTT
* [sonos2mqtt](https://github.com/svrooij/sonos2mqtt) - Control your [Sonos](https://www.sonos.com) speakers right from your mqtt server and get realtime updates about the tracks playing.
* [speedtest2mqtt](https://github.com/hobbyquaker/speedtest2mqtt) - Run speedtest-cli and publish results via MQTT.
* [unifi2mqtt](https://github.com/hobbyquaker/unifi2mqtt) - Publish connected clients from Ubiquiti Unifi to MQTT.
* [xiaomi2mqtt](https://github.com/svrooij/node-xiaomi2mqtt) - Bridge between Xiaomi Smart Home Gateway and MQTT. For relatively cheap Xiaomi Aqara zigbee sensors.

### Logic, Visualization, Logging

* [HOMR-REACT](https://github.com/klauserber/homr-react) - A configurable web visualization.
* [homely](https://github.com/baol/homely) - Early alpha stage collection of Go bridges and wiring for MQTT. Telegram and Desktop notification, Geofencing, Simple wiring logic, Interface with Arduino, integration with Domoticz (rerouting Domoticz mqtt messages to more descriptive topics)
* [jim](https://github.com/nathanielc/jim) - A programmable home automation assistant that provides a DSL for working with devices exposed via the mqtt-smarthome architecture.
* [logic4mqtt](https://github.com/owagner/logic4mqtt) - Logic and scripting engine for use with MQTT. Uses Java's general scripting interface, so scripts can be written in a multitude of languages like Javascript, Groovy etc.
* [mqtt-scripts](https://github.com/hobbyquaker/mqtt-scripts) - Logic and scripting engine for use with MQTT. Node.js based, require command works as expected.
* [influx4mqtt](https://github.com/hobbyquaker/influx4mqtt) - Insert incoming MQTT values into InfluxDB.
* [mqtt-admin](https://github.com/hobbyquaker/mqtt-admin) - MQTT Web Frontend: Publish, Subscribe and see Topic Status in a comfortable UI
* [mqtt-lightify](https://github.com/dzerrenner/mqtt-lightify) - Connect locally to an Osram lightify bridge via TCP. 

## Software (maybe) otherwise useable

A list of software that is maybe useful, even if it doesn't follow the mqtt-smarthome architecture can be found here: https://github.com/hobbyquaker/awesome-mqtt

## Additions and corrections

Additions and corrections are welcome! Please open an issue on GitHub or send a pull-request.
