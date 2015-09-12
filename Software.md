mqtt-smarthome
==============

_Note that this project is not associated with or endorsed by http://mqtt.org_

Home: https://github.com/mqtt-smarthome

## List of Software written with this proposal in mind
---------------------------

|------------------------------------------------------------------------    |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  |
|     	                                                                     |     **Hardware Interfaces**                                                                 	                                                                                                                                                                                            |
| [hm2mqtt](https://github.com/owagner/hm2mqtt) | Interface between EQ-3's Homematic line of smarthome devices and MQTT. |
| [knx2mqtt](https://github.com/owagner/knx2mqtt) | Interface between the KNX home automation standard and MQTT. Uses the Calimero KNX library. |
| [onkyo2mqtt](https://github.com/owagner/onkyo2mqtt) | Interface between Onkyo AVR's EISCP network remote protocol and MQTT. Uses the onkyo-eiscp library. |
| [hue2mqtt](https://github.com/owagner/hue2mqtt) | Interface between the Philips Hue bridge and MQTT. |
| [eno2mqtt](https://github.com/owagner/eno2mqtt) | Interface between an Enocean USB300 (TCM310) adapter and MQTT. |
| [modbus2mqtt](https://github.com/owagner/modbus2mqtt) | Modbus master which publishes register values via MQTT. |
| [cul2mqtt](https://github.com/hobbyquaker/cul2mqtt) | Interface between [Busware CUL](http://shop.busware.de/product_info.php/cPath/1/products_id/29) (FS20, HMS, EM, ...) and MQTT. |
| [bcontrol2mqtt](https://github.com/hobbyquaker/bcontrol2mqtt) | Publish values from [TQ Energy Manager](http://www.tq-group.com/produkte/produktdetail/prod/energy-manager/extb/Main/) to MQTT. |
| [mqtt-dmx-sequencer](https://github.com/hobbyquaker/mqtt-dmx-sequencer) | Control DMX devices via Art-Net by MQTT |

|     	                                                                     |     **Logic, Visualization, Logging**                                                                 	                                                                                                                                                                                            |
| [logic4mqtt](https://github.com/owagner/logic4mqtt) | Logic and scripting engine for use with MQTT. Uses Java's general scripting interface, so scripts can be written in a multitude of languages like Javascript, Groovy etc. |
| [influx4mqtt](https://github.com/hobbyquaker/influx4mqtt) | Insert incoming MQTT values into InfluxDB. |
|     	                                                                     |     **Misc**                                                                 	                                                                                                                                                                                            |
| [kodi2mqtt](https://github.com/owagner/kodi2mqtt) | Interface between a Kodi mediacenter instance and MQTT. |

## List of Software (maybe) otherwise useable
---------------------------
|------------------------------------------------------------------------    |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------  |
| [Node-RED](http://nodered.org/) | A visual tool for wiring the Internet of Things | 
| [mqtt2graphite](https://github.com/jpmens/mqtt2graphite) | Subscribe to MQTT topics and push to Graphite's Carbon server |
| [mqtt-panel](https://github.com/fabaff/mqtt-panel) | A web interface for MQTT |
| [Owntracks](http://owntracks.org/) |  Location tracking and geofencing for MQTT |
| [mqtt-dss-bridge](https://github.com/cgHome/mqtt-dss-bridge) | MQTT digitalSTROM-Server Bridge |
| [fritz2mqtt](https://github.com/akentner/fritz2mqtt) | Connect FRITZ!Box to MQTT |
| [agi-mqtt](https://github.com/jpmens/agi-mqtt) | Interface between Asterisk and MQTT |
| [mqtt-google-calendar](https://github.com/denschu/mqtt-google-calendar) | Google Calendar to MQTT bridge |
| [mqtt-os-status](https://github.com/oskarhagberg/mqtt-os-status) | Operating-system related data, published to an MQTT broker at fixed intervals. |

  
Smarthome-Software with MQTT adapters
---------------------------
* [ccu.io](https://github.com/hobbyquaker/ccu.io) has a MQTT adapter since V1.0.49
* [fhem](http://fhem.de/) has a [MQTT module](http://fhem.de/commandref.html#MQTT) since V5.6 
* [ioBroker](https://github.com/ioBroker) has a [MQTT adapter](https://github.com/ioBroker/ioBroker.mqtt)
* [openhab](https://github.com/openhab)
  has a [MQTT binding](https://github.com/openhab/openhab/wiki/MQTT-Binding)

Additions and corrections
-------------------------
Additions and corrections are welcome! Please open an issue on GitHub, send a 
pullrequest or simply e-mail me at owagner AT tellerulam.com.
