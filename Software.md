# Software for mqtt-smarthome

Software written with the mqtt-smarthome convention in mind and actively maintained. The list of
2015–2018, unchanged, is [legacy/Software.md](legacy/Software.md); entries from it are kept here
when their project is still maintained and in use.

The **spec** column names the [SPEC.md](SPEC.md) version a project announces; `—` means it follows
the 2015 convention ([legacy/Architecture.md](legacy/Architecture.md)).

If your software follows the convention, you are welcome to add this badge to its README:
[![mqtt-smarthome](https://img.shields.io/badge/mqtt-smarthome-blue.svg)](https://github.com/mqtt-smarthome/mqtt-smarthome)

```markdown
[![mqtt-smarthome](https://img.shields.io/badge/mqtt-smarthome-blue.svg)](https://github.com/mqtt-smarthome/mqtt-smarthome)
```

## Interfaces

| Project                                                               | Connects                                                               | Spec |
| --------------------------------------------------------------------- | ---------------------------------------------------------------------- | ---- |
| [alexa-remote-mqtt](https://github.com/hobbyquaker/alexa-remote-mqtt) | Amazon Echo devices                                                    | 2.0  |
| [cul2mqtt](https://github.com/hobbyquaker/cul2mqtt)                   | Busware CUL: FS20, EM, HMS, S300TH, FHT, MAX!, …                       | 2.0  |
| [ecoflow2mqtt](https://github.com/hobbyquaker/ecoflow2mqtt)           | EcoFlow STREAM and PowerStream micro-inverters                         | 2.0  |
| [fritz2mqtt](https://github.com/hobbyquaker/fritz2mqtt)               | AVM FRITZ!Box routers: DSL/WAN, traffic, WLAN, LAN hosts, call monitor | 2.0  |
| [govee2mqtt](https://github.com/hobbyquaker/govee2mqtt)               | Govee WiFi lights (LAN API)                                            | 2.0  |
| [helios2mqtt](https://github.com/mreschka/helios2mqtt)                | Helios EasyControls ventilation units (Modbus TCP)                     | —    |
| [hm2mqtt.js](https://github.com/hobbyquaker/hm2mqtt.js)               | Homematic and Homematic IP through a CCU                               | 2.0  |
| [homeconnect2mqtt](https://github.com/hobbyquaker/homeconnect2mqtt)   | BSH Home Connect appliances (Bosch, Siemens, Neff, Gaggenau, …)        | 2.0  |
| [lgsb2mqtt](https://github.com/hobbyquaker/lgsb2mqtt)                 | LG soundbars                                                           | 2.0  |
| [lgtv2mqtt](https://github.com/hobbyquaker/lgtv2mqtt)                 | LG webOS smart TVs                                                     | 2.0  |
| [mqttpc](https://github.com/hobbyquaker/mqttpc)                       | processes on a host                                                    | 2.0  |
| [sonos2mqtt](https://github.com/svrooij/sonos2mqtt)                   | Sonos speakers                                                         | —    |
| [speedtest2mqtt](https://github.com/hobbyquaker/speedtest2mqtt)       | internet speed tests                                                   | 2.0  |
| [unifi2mqtt](https://github.com/hobbyquaker/unifi2mqtt)               | Ubiquiti UniFi network controllers: client presence, devices, WLANs    | 2.0  |
| [wiim2mqtt](https://github.com/hobbyquaker/wiim2mqtt)                 | WiiM (LinkPlay) audio streamers                                        | 2.0  |

Libraries for writing interfaces:
[mqtt-interfaces-core](https://github.com/hobbyquaker/mqtt-interfaces-core) (Node.js) — every
hobbyquaker interface above is built on it, and
[Smart Home Engine ("she")](https://github.com/hobbyquaker/she) can install, configure and manage
them.

## Recording

| Project                                                                 | Records into  | Spec |
| ----------------------------------------------------------------------- | ------------- | ---- |
| [influx4mqtt](https://github.com/hobbyquaker/influx4mqtt)               | InfluxDB      | 2.0  |
| [mqtt2elasticsearch](https://github.com/hobbyquaker/mqtt2elasticsearch) | Elasticsearch | 2.0  |

## Logic

| Project                                                         | What it is                                                                                             |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| [Smart Home Engine ("she")](https://github.com/hobbyquaker/she) | scripts in plain JavaScript, MQTT and Matter, a web IDE; manages mqtt-interfaces-core based interfaces |
| [dan](https://github.com/nathanielc/dan)                        | a home automation programming language with native MQTT support (formerly jim)                         |

## User interface

| Project                                    | What it is                                     |
| ------------------------------------------ | ---------------------------------------------- |
| [feezal](https://github.com/feezal/feezal) | visually build MQTT-driven apps and dashboards |

## Works well alongside

Not written for mqtt-smarthome, but popular projects that share a broker with it nicely:
[zigbee2mqtt](https://www.zigbee2mqtt.io), [ESPHome](https://esphome.io),
[Node-RED](https://nodered.org) and [Home Assistant](https://www.home-assistant.io).

## Additions

Additions are welcome — open an issue or a pull request. An entry needs a project that follows the
convention, is maintained and is used by more people than its author.
