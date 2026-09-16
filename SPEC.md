# mqtt-smarthome specification 2.0

_Status: 2.0, 2026-09-16. Written from what
[mqtt-interfaces-core](https://github.com/hobbyquaker/mqtt-interfaces-core) 0.16 and the adapters
built on it implement. Sections marked **reserved** describe things the convention defines but no
2.x implementation provides yet._

This is the successor of [legacy/Architecture.md](legacy/Architecture.md) (V0.5, 2015). That
document stays as the historical reference; where the two disagree, this one wins for anything that
announces spec `2.x`.

_Not associated with or endorsed by https://mqtt.org_

Contents: [1 Scope](#1-scope) · [2 Versioning](#2-versioning-and-conformance) ·
[3 Topics](#3-topic-structure) · [4 Retain and QoS](#4-retain-and-qos) ·
[5 Payloads](#5-payloads) · [6 Introspection](#6-introspection-nameinfo) ·
[7 Maintenance](#7-maintenance-topics) ·
[8 Interface description](#8-interface-description-home-assistant-mqtt-discovery) ·
[9 CLI and environment](#9-cli-and-environment-conventions-recommended) ·
[10 Device discovery](#10-device-discovery-recommended) · [11 Logging](#11-logging-recommended) ·
[12 Migration](#12-migration-from-the-2015-convention-v05) · [13 History](#13-history)

## 1. Scope

mqtt-smarthome is a **convention**, not a protocol: it says how a program that bridges some hardware
or service — an "interface", "adapter" or `xyz2mqtt` — presents itself on an MQTT broker, so that
logic engines, dashboards, management tools and Home Assistant can use every adapter the same way.

It deliberately does **not** abstract devices into higher-level concepts ("a light", "a cover").
Every adapter exposes what its hardware has, as items; abstraction, if needed, happens above the bus
— or through the interface description (section 8), which maps items to Home Assistant entity types
for the consumers that want it.

The convention is independent of any language or library. An implementation states the version it
implements (section 2); mqtt-interfaces-core is one such implementation, for Node.js.

The words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are used as in
[RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## 2. Versioning and conformance

- The spec has a version `MAJOR.MINOR`. Implementations state the version they implement:
  libraries in their documentation, adapters in `<name>/info` (section 6).
- MINOR versions only add; MAJOR versions may change meaning.
- An adapter conforms to 2.0 if it implements sections 3, 4, 5 and 6. Sections 7 and 8 are
  optional and recommended; sections 9–11 are recommendations. Which of the optional parts an
  instance has switched on is visible in `<name>/info` and in the retained topics it publishes.

## 3. Topic structure

```
<name>/<function>/<item...>
```

- `<name>` — the instance name. Identifies one running adapter instance. MUST be configurable
  (conventionally `--name`/`-n`), so several instances of the same adapter can share a broker.
  Defaults to a short adapter-specific word (`lgtv`, `soundbar`, `cul`, …). MUST NOT contain `/`,
  `+` or `#`. It SHOULD NOT be a serial number: the user names an instance, not the factory.
- `<function>` — one of `connected`, `status`, `set`, `get`, `info`, `maintenance`. Adapters MAY
  define additional functions (`raw`, `paramset`, `rpc`); they SHOULD be documented in the
  adapter's README and MUST NOT collide with the reserved ones.
- `<item...>` — one or more levels defined by the adapter (`volume`, `play/position`,
  `fs20/1a2b01/state`, `client/<mac>/present`). Item names SHOULD be `snake_case` and stable across
  versions, and MUST be identical under `status`, `set` and `get`. Levels MUST NOT be empty.

Characters from device names that are not valid in topics (`/`, `+`, `#`) MUST be replaced
(recommended: `_`); spaces SHOULD be avoided in item names.

### 3.1 `<name>/connected`

Retained. An integer enum as a plain value:

| value | meaning                                                              |
| ----- | -------------------------------------------------------------------- |
| `0`   | adapter not running / not connected to the broker                    |
| `1`   | connected to the broker, hardware not reachable (off, not paired, …) |
| `2`   | connected to the broker and to the hardware, fully operational       |

- `0` MUST be set by the MQTT Last Will and Testament and SHOULD also be published explicitly on
  graceful shutdown.
- `1` and `2` MUST be published on every transition and on every (re)connect to the broker.
- An adapter that bridges several devices under one `<name>` publishes `2` when the bridge or
  service itself is reachable; the reachability of a single device behind it goes into a status
  item (`<name>/status/<device>/online`).
- An adapter without hardware of its own — a sink that writes other adapters' traffic into a
  database, a process controller — publishes `2` when its target (the database, the host) is
  usable.
- Change from 2015: `connected` is always a plain value, never JSON.

### 3.2 `<name>/status/<item>`

Published by the adapter: sensor values, actuator feedback, device information.

- **Retain**: persistent state (a volume, a temperature, a client's presence) MUST be retained;
  one-shot events (a button press, a spoken command, an incoming call) MUST NOT be retained. Values
  that change constantly and are worthless when stale (a play position) SHOULD NOT be retained.
- Published on start (the full state as soon as it is known), on every change reported by the
  hardware — including changes made by other means, such as the device's own remote or app — and
  again after every broker reconnect, so the broker's retained state is complete. Events are not
  re-published.
- An adapter SHOULD publish only when the value or its timestamp meaningfully changed; it MUST NOT
  republish unchanged state in a tight loop.
- Items that no longer apply (a client that left, a device removed from a bridge) SHOULD be cleared
  by publishing an empty retained payload. This includes items a **previous run** left retained: an
  adapter whose item set is dynamic SHOULD read back its retained `<name>/status/#` on start and
  clear what its first complete picture no longer contains.
- An adapter without items of its own (a sink) publishes no `status` at all.

### 3.3 `<name>/set/<item>`

Published by consumers to request a change. Same item hierarchy as `status`. MUST NOT be retained.
The adapter

- MUST accept plain values and `{"val": …}` JSON transparently (section 5.3),
- SHOULD validate and clamp values (ranges, enums), and MUST log a rejected or failed request at
  `warn` with its topic and payload,
- MUST NOT echo the request as a status; the status follows from the hardware's feedback or, when
  the hardware gives none, from the adapter's own state after the action succeeded.

Additional levels below the item MAY carry parameters (`set/<device>/speak`,
`set/light/1/brightness`).

Adapters that accept arbitrary protocol-level commands (`set/raw`) MUST keep that behind an opt-in
option that is off by default: a raw transmitter is a security surface.

### 3.4 `<name>/get/<item>` — reserved

From 2015: a message with any payload requests an active read of `<item>` from the hardware; the
result is published as `status`. MUST NOT be retained. No 2.x implementation provides it yet; an
adapter that does documents it in its README.

### 3.5 `<name>/command` — dropped

The 2015 free-form command channel is not part of 2.x. Use `set/<item>` with parameter levels, or an
adapter-specific function (section 3), instead.

## 4. Retain and QoS

- Retain as in 3.1–3.4. Rule of thumb: "would a consumer that starts now want to see this?" →
  retain.
- QoS 0 for everything. QoS 1 is discouraged — a duplicate `set` repeats a hardware action, a
  duplicate `status` repeats an event in the logic layer; QoS 2 is unnecessary on a local bus.

## 5. Payloads

### 5.1 Plain values

A plain value is the UTF-8 text of a boolean (`true`/`false`), a number (`12`, `-3.5`) or a string,
without quotes. Lists and structured values are JSON (`["HDMI_1","HDMI_2"]`,
`[{"id":"netflix","title":"Netflix"}]`). An empty payload means "no value" (and clears a retained
topic).

### 5.2 JSON status objects

A status MAY instead be a JSON object with at least `val`:

```json
{ "val": 17.9, "ts": 1421792677000, "lc": 1421792677000 }
```

| field | type | meaning                                                          |
| ----- | ---- | ---------------------------------------------------------------- |
| `val` | any  | the value as it would be published plain (required)              |
| `ts`  | int  | when the value was obtained, ms since the epoch                  |
| `lc`  | int  | when the value last **changed**, ms since the epoch (`lc <= ts`) |

- `ts` and `lc` MUST both be present when the object form is used. An adapter whose hardware reports
  its own time for a value SHOULD use it for `ts` and `lc`.
- Adapters MAY add interface-specific fields. They MUST NOT redefine `val`, `ts` or `lc`, and
  SHOULD group their fields under one key named after the adapter or protocol (`"hm": {…}`,
  `"unifi": {…}`), so consumers can tell them from future spec fields:

  ```json
  {
    "val": true,
    "ts": 1421792677000,
    "lc": 1421792677000,
    "unifi": { "ap_mac": "aa:bb:cc:dd:ee:ff", "rssi": -61 }
  }
  ```

- An instance MUST use one form for all its `status` items, so consumers never guess per item. The
  form is selectable per instance (section 9). A consumer recognises the object form by a JSON
  object that has a `val` key.
- **The object form is the default.** Adapters SHOULD publish `{val, ts, lc}` unless configured
  otherwise: `ts` and `lc` are the one thing a consumer cannot reconstruct from a retained plain
  value (the age of a reading, "open since"), and the interface description (section 8) hides the
  difference from Home Assistant. The plain form is the opt-out for consumers that cannot parse
  JSON.

### 5.3 `set` payloads

Consumers MAY send a plain value or `{"val": …}`; adapters MUST accept both and unwrap `val`. Other
JSON objects and arrays are passed to the item's handler as structured parameters (for example
`{"id": "…", "params": {…}}` for an app launch).

Conversions adapters SHOULD apply:

- booleans: `true`/`false`, `1`/`0`, `on`/`off`, `yes`/`no`, case-insensitive
- numbers: decimal text, rounded and clamped to the item's range
- enums: case-insensitive name match; a numeric index MAY be accepted

## 6. Introspection: `<name>/info`

Retained JSON describing the running instance, published on every broker connect, so tools find
every adapter on a broker by subscribing to `+/info`:

| field         | requirement | example / meaning                                      |
| ------------- | ----------- | ------------------------------------------------------ |
| `name`        | MUST        | adapter (package) name, `lgtv2mqtt`                    |
| `version`     | MUST        | adapter version, `3.0.6`                               |
| `spec`        | MUST        | spec version implemented, `2.0`                        |
| `node`        | SHOULD      | runtime version; other runtimes use their own key      |
| `host`        | SHOULD      | host name                                              |
| `pid`         | SHOULD      | process id                                             |
| `started`     | SHOULD      | ISO 8601 start time (uptime is derivable)              |
| `maintenance` | SHOULD      | whether the maintenance topics are enabled (section 7) |

Adapters MAY add fields about the instance (`"tv": "192.168.1.20"`, `"mode": "cloud"`); they MUST
NOT redefine the fields above. `info` is small and static — it is not a status topic. Clearing it on
shutdown is not required: a consumer combines `info` with `connected` to see liveness.

## 7. Maintenance topics

Optional and recommended. When implemented they MUST be disableable (conventionally
`--no-maintenance`), and `info.maintenance` says whether they are active.

| topic                             | retained | payload                                | effect                                                                 |
| --------------------------------- | -------- | -------------------------------------- | ---------------------------------------------------------------------- |
| `<name>/maintenance/set/loglevel` | no       | `error` \| `warn` \| `info` \| `debug` | change the log level at runtime (not persisted)                        |
| `<name>/maintenance/set/restart`  | no       | any                                    | graceful shutdown (`connected` → `0`), exit 0; the supervisor restarts |
| `<name>/maintenance/stats`        | yes      | JSON, see below                        | published by the adapter at a configurable interval                    |

`maintenance/stats` carries process statistics for dashboards: `rss`, `heapUsed`, `heapTotal`
(bytes), `cpu` (percent of one core over the interval), `eventLoopLag` (worst delay in ms over the
interval), `uptime` (s) and `ts` (ms since the epoch). Runtimes without a heap or an event loop leave
those fields out. The interval is configurable; `0` switches the topic off.

Unknown maintenance topics and commands are logged at `warn` and ignored. The restart works as
intended only under a supervisor that restarts a process after a clean exit (a systemd unit with
`Restart=always`, a container with `--restart unless-stopped`).

**Security.** These topics let anyone who may publish on the broker restart an adapter or raise its
log level. Brokers SHOULD use authentication and per-client ACLs (an adapter needs `<name>/#` and
the discovery prefix, nothing else); an adapter on a broker that cannot be secured SHOULD run with
the maintenance topics disabled.

## 8. Interface description (Home Assistant MQTT discovery)

2.x adopts **Home Assistant's MQTT discovery in its device-based form** (Home Assistant 2024.11 and
later) as the machine-readable description of what an adapter instance offers: its entities, their
types, ranges and units, the device information and the availability. mqtt-smarthome topics stay
the wire protocol; the discovery payload is the schema on top of it. It is the only widely deployed,
documented entity-description format, so the convention uses it instead of inventing one.

Home Assistant is one consumer among others. Dashboards build widgets from the same payloads (a
`number` with `min`/`max` becomes a slider, a `select` a drop-down), management tools list an
instance's entities without adapter-specific code, and other systems that read the format use it
as well. Nothing in an adapter depends on Home Assistant being present, and adapters do not
implement Home Assistant features that have no mqtt-smarthome meaning.

The payload format is Home Assistant's
([device discovery payload](https://www.home-assistant.io/integrations/mqtt/#device-discovery-payload));
abbreviated keys are allowed. On top of it:

- **Topic**: `<ha-prefix>/device/<id>/config`, retained. `<ha-prefix>` is configurable and defaults
  to `homeassistant`. `<id>` is `<adapter>_<name>` with every character outside `[A-Za-z0-9_-]`
  replaced by `_`. An adapter that bridges several devices publishes **one config per device**,
  `<id>` = `<adapter>_<name>_<device>`, each linked to the bridge's device with `via_device`.
- **On by default** and disableable (conventionally `--no-ha-discovery`). Disabling it MUST also
  clear (empty retained payload) the configs the instance published before.
- **Published** on every broker connect and whenever the entity set changes (a model is learned, an
  input list or a device appears). A device that disappears from a bridge gets its config cleared.
- **Entities**: `state_topic` is `<name>/status/<item>`, `command_topic` is `<name>/set/<item>`.
  With JSON status (5.2) an entity carries `value_template: "{{ value_json.val }}"`. `unique_id` is
  `<id>_<item>` with `/` replaced by `_`. Stateless platforms (`button`, `notify`) have no state
  topic.
- **Availability** comes from `<name>/connected`: available when the value is at least `2` (an
  adapter MAY use `1` for entities that work without the device, e.g. a wake-on-LAN power switch).
  An entry of the `availability` list carries only `topic`, `payload_available`,
  `payload_not_available` and `value_template` — **never** `availability_template` (`avty_tpl`),
  which belongs to the single-topic form. Home Assistant rejects the whole device payload over one
  unknown key in a list entry and creates none of its entities. A bridged device MAY add an entry
  of its own (`<name>/status/<device>/online`) next to the bridge's; with more than one entry,
  `availability_mode` is `all`.
- The payload SHOULD carry an `origin` block (`name`, `sw_version`, `support_url`) naming the
  adapter, and a device block with `identifiers`, `name` and whatever the adapter knows
  (`manufacturer`, `model`, `sw_version`, …). Entity names are short and device-relative ("Volume",
  not "Living room TV volume"); Home Assistant prefixes the device name.
- Where Home Assistant's entity model falls short (it has no MQTT media player), an adapter
  publishes the entities that do exist (switches, numbers, selects, sensors, buttons) rather than
  forking the format.

The payloads are easy to get subtly wrong and hard to check without a running Home Assistant, so an
implementation SHOULD build them with a pure function and test it; how it validates them against
Home Assistant's schema is its own concern.

## 9. CLI and environment conventions (recommended)

Not required by the convention, but implemented by the reference library and relied on by
management tools, which can then configure every adapter the same way.

- Option names are long-form kebab-case; short aliases are optional. Every option can also be set
  as the environment variable `<ADAPTER>_<OPTION>` (`LGTV2MQTT_MQTT_URL`); precedence is CLI >
  environment > default. There is no configuration file format of the adapter's own.
- The broker options additionally fall back to the unprefixed `MQTT_URL`, `MQTT_USERNAME`,
  `MQTT_PASSWORD` and `MQTT_TLS_CA`, so one file configures every adapter on a host. The client id
  prefix is per instance and has no shared variable.

| option                               | default            | meaning                                                  |
| ------------------------------------ | ------------------ | -------------------------------------------------------- |
| `-u`, `--mqtt-url` (`--url`)         | `mqtt://localhost` | broker URL incl. `mqtts://`, `user:pass@` allowed        |
| `--mqtt-username`, `--mqtt-password` |                    | broker credentials                                       |
| `--mqtt-client-id-prefix`            |                    | client id = `<prefix><name>_<random>`                    |
| `--mqtt-tls-ca`                      |                    | CA certificate file for `mqtts://`                       |
| `-n`, `--name`                       | adapter-specific   | instance name and topic prefix (section 3)               |
| `--json-payloads` / `--no-…`         | on                 | `{val, ts, lc}` status objects (5.2); off = plain values |
| `--ha-discovery` / `--no-…`          | on                 | section 8                                                |
| `--ha-prefix`                        | `homeassistant`    | discovery prefix                                         |
| `--maintenance` / `--no-…`           | on                 | section 7                                                |
| `--stats-interval`                   | `60`               | seconds between `maintenance/stats`; `0` disables        |
| `-v`, `--verbosity`                  | `info`             | `error` \| `warn` \| `info` \| `debug`                   |
| `--install` / `--uninstall`          |                    | install or remove the instance as a service (root)       |
| `--config-schema`                    |                    | print the JSON Schema of the instance configuration      |
| `--publish-raw`, `--raw-set`         | off                | protocol-level escape hatches, adapter-specific          |

An adapter that publishes no status (a sink) omits `--json-payloads`; one that announces no entities
omits `--ha-discovery` and `--ha-prefix`. Device address options are adapter-specific (`--address`,
`--tv`, `--serialport`, …); section 10 adds the value `auto` to them.

**`--config-schema`** prints a JSON Schema (draft 2020-12) of the instance configuration — one
property per option, keyed by the option name — so a management tool can render a form for any
adapter. Extension keywords on a property:

| keyword            | meaning                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| `x-env`            | the option's environment variable                                                               |
| `x-secret`         | `true`: a credential, to be masked                                                              |
| `x-file`           | the option holds the path of a file the user maintains: `{format, example, schema, describe}`   |
| `x-discover`       | the option that section 10 fills, with the kind of scan: `network`, `serial`, `cloud` or a list |
| `x-discover-needs` | the options the scan itself needs (the credentials of a cloud scan)                             |

The schema itself carries `x-adapter`: `{name, version, envPrefix}`. Options that only control a
single run (`--install`, `--uninstall`, `--config-schema`, `--discover…`) are not part of it.

Deviations in pre-2.x adapters that a port fixes: `--topic-prefix` → `--name`, `--verbose` →
`--verbosity`, single-letter-only options → long names with short aliases.

## 10. Device discovery (recommended)

Finding the device is the adapter's job, so an adapter used on its own offers it too. A management
tool runs the adapter's scan rather than scanning itself.

- `--discover` scans once, prints the devices found (address, name, model, how each was found) and
  exits; `--discover-json` prints the same as JSON for tools; `--discover-timeout <s>` bounds the
  scan. Mandatory options do not apply to a scan, except the ones the scan itself needs
  (`x-discover-needs`).
- `--discover-address <address|cidr>` probes one host, or sweeps a range, beyond the reach of
  multicast and broadcast — neither crosses a router.
- `auto` as the value of the device option (`--address auto`) scans at start. An adapter for one
  device takes the single match and **refuses to start** when there is none or more than one,
  listing every candidate at `warn` — it never guesses. An adapter that bridges every device of a
  kind (an array option) takes all matches and refuses only when there is none. A running adapter
  never scans continuously; an installer SHOULD resolve `auto` once and store the result.
- A network candidate carries its `address` and, only when they resolve back to that address, its
  `fqdn` and `hostname`; `auto` prefers the fully qualified name, and `--discover-ip` forces the
  address. A serial candidate is its stable device path (`/dev/serial/by-id/…`).

How an adapter declares what its devices look like (SSDP search target, DNS-SD service type, UDP
probe, TCP ports, MAC prefixes, a vendor cloud) is up to the implementation; mqtt-interfaces-core
documents its hint format in its README.

## 11. Logging (recommended)

Distilled from years of support requests for adapters and their device libraries.

- **Levels** `error` > `warn` > `info` > `debug`:
  - `error` only for things a human must fix: bad configuration, broker authentication, a failed
    pairing or certificate check.
  - `warn` for an unreachable or misbehaving device, a rejected `set` and a maintenance action —
    and for every "action required" message (pair on the device, accept a certificate, mount a
    volume), naming the file, option or path involved.
  - `info` for the lifecycle: starting, connected, subscribed, discovery published, shutting down.
  - `debug` for attempts and raw traffic.
- **An unreachable device is normal operation** (a TV in standby): `warn`, not `error`. Log the
  transition (reachable → unreachable) once, suppress repeats of the identical error, and log the
  recovery at `info` with how long the device was away.
- **Attempts at `debug`, outcomes at `warn`.** When an adapter tries several ways to reach a device
  (TLS, then plain), each attempt is `debug`; only the failure of all of them becomes one `warn`
  line that lists every candidate with its error.
- **Never swallow an error from the device.** A device library returns errors with a stable code and
  the device's own reason; the adapter logs every rejected `set` with topic, payload and that
  reason. A `set` that silently does nothing is a bug.
- **Security checks log enough to act on**: a failed certificate pin logs the presented
  fingerprints; a first contact that is trusted on first use logs what was stored.
- **Raw traffic** goes to `debug` with direction prefixes: `mqtt >` (published), `mqtt <`
  (received), `<device> >` and `<device> <`. When the device closes the connection, log it together
  with the last request sent: firmware that drops a connection in response to a request is
  diagnosable that way.
- **`connected` transitions** are logged, so a user can correlate them with what the device did.
- The runtime log level (`maintenance/set/loglevel`) is the way to get `debug` output without a
  restart.
- **Under systemd** (journald, detected by `JOURNAL_STREAM`) log lines carry the syslog priority
  prefix `<N>` and no timestamp of their own, and the unit's `SyslogIdentifier` names the instance
  (`<adapter>@<name>`), so `journalctl -p warning -u <adapter>@<name>` works. An environment
  variable (`<ADAPTER>_LOG_FORMAT=journal|text`) forces either format.

## 12. Migration from the 2015 convention (V0.5)

| 2015                                         | 2.0                                                                   |
| -------------------------------------------- | --------------------------------------------------------------------- |
| `connected` could be JSON                    | always plain `0`/`1`/`2`                                              |
| `status` plain or JSON, the adapter's choice | both valid; JSON is the default, switchable per instance              |
| extra JSON fields prefixed (`knx_src_addr`)  | extra fields grouped under one key (`"knx": {…}`)                     |
| `set` always plain                           | plain, `{"val": …}` or structured JSON                                |
| `get/`                                       | reserved, unchanged meaning                                           |
| `command`                                    | dropped                                                               |
| —                                            | `info`, `maintenance/`, the interface description, CLI/env, discovery |
| topic prefix should be configurable          | must be; `--name`                                                     |
| item names free-form (spaces, device names)  | `snake_case` recommended; invalid characters replaced                 |

An adapter moving to 2.x makes a clean break with a major version bump and does not keep the old
topics behind a compatibility option. Its README documents the old and the new topics side by side.

## 13. History

- V0.1–V0.5, January 2015 — Oliver Wagner (owagner), see
  [legacy/Architecture.md](legacy/Architecture.md).
- 2.0, 2026-09-16 — written from mqtt-interfaces-core and the adapters built on it (first draft
  2026-08-22). Adds `info`, the maintenance topics, Home Assistant discovery as the interface
  description, and conventions for CLI/environment, device discovery and logging; makes the JSON
  status object the default; drops `command`; pins `connected` to plain values.
