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

[Software.md](Software.md) lists software written for mqtt-smarthome that is maintained today. One
way to build an mqtt-smarthome, maintained by hobbyquaker — any other software that follows the
convention fits in the same way:

- [mqtt-interfaces-core](https://github.com/hobbyquaker/mqtt-interfaces-core): core library for
  writing `xyz2mqtt` interfaces in Node.js
- Interfaces:
  [alexa-remote-mqtt](https://github.com/hobbyquaker/alexa-remote-mqtt) (Amazon Echo),
  [cul2mqtt](https://github.com/hobbyquaker/cul2mqtt) (Busware CUL: FS20, EM, HMS, FHT, MAX!, …),
  [ecoflow2mqtt](https://github.com/hobbyquaker/ecoflow2mqtt) (EcoFlow micro-inverters),
  [fritz2mqtt](https://github.com/hobbyquaker/fritz2mqtt) (AVM FRITZ!Box),
  [govee2mqtt](https://github.com/hobbyquaker/govee2mqtt) (Govee WiFi lights),
  [hm2mqtt.js](https://github.com/hobbyquaker/hm2mqtt.js) (Homematic CCU),
  [homeconnect2mqtt](https://github.com/hobbyquaker/homeconnect2mqtt) (BSH Home Connect appliances),
  [lgsb2mqtt](https://github.com/hobbyquaker/lgsb2mqtt) (LG soundbars),
  [lgtv2mqtt](https://github.com/hobbyquaker/lgtv2mqtt) (LG webOS TVs),
  [mqttpc](https://github.com/hobbyquaker/mqttpc) (processes on a host),
  [speedtest2mqtt](https://github.com/hobbyquaker/speedtest2mqtt) (internet speed tests),
  [unifi2mqtt](https://github.com/hobbyquaker/unifi2mqtt) (Ubiquiti UniFi),
  [wiim2mqtt](https://github.com/hobbyquaker/wiim2mqtt) (WiiM/LinkPlay streamers) — all on
  mqtt-interfaces-core; [Smart Home Engine ("she")](https://github.com/hobbyquaker/she) can manage
  them
- Recording: [influx4mqtt](https://github.com/hobbyquaker/influx4mqtt) (InfluxDB),
  [mqtt2elasticsearch](https://github.com/hobbyquaker/mqtt2elasticsearch) (Elasticsearch)
- Automation: [Smart Home Engine ("she")](https://github.com/hobbyquaker/she), Node-RED, Home Assistant, …
- User interface: [feezal](https://github.com/feezal/feezal), Node-RED-Dashboard, Home Assistant, any MQTT-driven apps and dashboards

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
