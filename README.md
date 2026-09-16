# mqtt-smarthome

A convention for using MQTT as the central message bus of a smart home.

_Not associated with or endorsed by https://mqtt.org_

## The convention

Every bridge to hardware or a service — an _interface_, often called `xyz2mqtt` — presents itself on
the broker in the same way:

```
<name>/connected          0, 1 or 2: interface down, up without its device, fully operational
<name>/status/<item>      the state, retained: {"val": 21.5, "ts": …, "lc": …}
<name>/set/<item>         requests a change
<name>/info               what is running there: name, version, spec version
```

The current version is **[SPEC.md](SPEC.md), mqtt-smarthome 2.0**. It covers topics, retain rules,
payloads, the `info` and maintenance topics, Home Assistant discovery as the interface description,
and recommended conventions for command line options, device discovery and logging. The original
proposal of 2015 is [legacy/Architecture.md](legacy/Architecture.md).

## The idea

mqtt-smarthome is not a piece of software but a concept: a small set of rules for how a bridge to
hardware or a service presents itself on an MQTT broker. Everything around the broker stays
decoupled, and every part can be replaced without touching the others. Popular projects like
[zigbee2mqtt](https://www.zigbee2mqtt.io), [ESPHome](https://esphome.io),
[Node-RED](https://nodered.org) and [Home Assistant](https://www.home-assistant.io) show how well
this works on a shared broker.

- **Interfaces**: any program that follows the convention. The `xyz2mqtt` adapters built on
  [mqtt-interfaces-core](https://github.com/hobbyquaker/mqtt-interfaces-core) run standalone;
  [Smart Home Engine ("she")](https://github.com/hobbyquaker/she) can manage them.
- **Automation**: [Smart Home Engine ("she")](https://github.com/hobbyquaker/she), Node-RED, Home
  Assistant or anything else that speaks MQTT, entirely up to the user.
- **User interface**: [feezal](https://github.com/feezal/feezal), Node-RED Dashboard, Home
  Assistant, … One of them, a different one later, or several in parallel.

## Home Assistant

Every interface describes what it offers — its entities, their types, ranges and units — with Home
Assistant's MQTT discovery. It is the de-facto standard for describing entities over MQTT, so
mqtt-smarthome adopts it instead of inventing a format of its own: Home Assistant picks the
interfaces up without any configuration, and dashboards and management tools read the same
payloads. Home Assistant is one consumer among others; nothing in an interface depends on it, and
everything works on a bare broker. Details in [SPEC.md section 8](SPEC.md#8-interface-description-home-assistant-mqtt-discovery).

## Software

Software written with the mqtt-smarthome convention in mind and actively maintained. The
hobbyquaker projects below are one way to build an mqtt-smarthome — any other software that follows
the convention fits in the same way. The list of 2015–2018, unchanged, is
[legacy/Software.md](legacy/Software.md); entries from it are listed here when their project is
still maintained and in use.

The **spec** column names the [SPEC.md](SPEC.md) version a project announces; `—` means it follows
the 2015 convention ([legacy/Architecture.md](legacy/Architecture.md)).

If your software follows the convention, you are welcome to add this badge to its README:
[![mqtt-smarthome](https://img.shields.io/badge/mqtt-smarthome-blue.svg)](https://github.com/mqtt-smarthome/mqtt-smarthome)

```markdown
[![mqtt-smarthome](https://img.shields.io/badge/mqtt-smarthome-blue.svg)](https://github.com/mqtt-smarthome/mqtt-smarthome)
```

### Interfaces

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

### Recording

| Project                                                                 | Records into  | Spec |
| ----------------------------------------------------------------------- | ------------- | ---- |
| [influx4mqtt](https://github.com/hobbyquaker/influx4mqtt)               | InfluxDB      | 2.0  |
| [mqtt2elasticsearch](https://github.com/hobbyquaker/mqtt2elasticsearch) | Elasticsearch | 2.0  |

### Logic

| Project                                                         | What it is                                                                                             |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| [Smart Home Engine ("she")](https://github.com/hobbyquaker/she) | scripts in plain JavaScript, MQTT and Matter, a web IDE; manages mqtt-interfaces-core based interfaces |
| [dan](https://github.com/nathanielc/dan)                        | a home automation programming language with native MQTT support (formerly jim)                         |

### User interface

| Project                                    | What it is                                     |
| ------------------------------------------ | ---------------------------------------------- |
| [feezal](https://github.com/feezal/feezal) | visually build MQTT-driven apps and dashboards |

### Works well alongside

Not written for mqtt-smarthome, but popular projects that share a broker with it nicely:
[zigbee2mqtt](https://www.zigbee2mqtt.io), [ESPHome](https://esphome.io),
[Node-RED](https://nodered.org) and [Home Assistant](https://www.home-assistant.io).

### Additions

Additions are welcome — open an issue or a pull request. An entry needs a project that follows the
convention, is maintained and is used by more people than its author.

## History

mqtt-smarthome started in December 2014 as a convention for using MQTT as the central message bus of
a smart home. Oliver Wagner wrote the initial specification, published as
[Architecture.md](legacy/Architecture.md) in January 2015; Sebastian Raff
([hobbyquaker](https://github.com/hobbyquaker)) was involved from the beginning and built adapters
and tools on it.

In 2026, as part of a modernization of hobbyquaker's smart home software fleet (background in the
[she README](https://github.com/hobbyquaker/she)), the convention was picked up again. The adapters
now share a core library and describe themselves via Home Assistant discovery, and the convention
got its 2.0 revision.

## The documents of 2015–2018

Kept unchanged in [legacy/](legacy/):

- [Architecture.md](legacy/Architecture.md) — the architectural proposal, V0.1–V0.5
- [Software.md](legacy/Software.md) and [Devices.md](legacy/Devices.md) — the lists of that time
- [Getting started with mqtt-smarthome — Homematic and Node-RED](legacy/howtos/homematic.md)
- [MQTT Smarthome Primer](legacy/talks/MQTT%20Smarthome%20Primer%20-%20EQ-3%20Homematic%20Conference%20-%20April%202015%20Kassel.pdf) — the talk at the eQ-3 Homematic conference, Kassel, April 2015
- [README.md](legacy/README.md) — the previous version of this page

## Feedback

Questions, proposals for the specification and additions to the software list are welcome as
issues or pull requests.

## License

[MIT](LICENSE)
