# mqtt-smarthome specification 2.0 — DRAFT

_Status: draft, 2026-08-22. Written from what
[mqtt-interfaces-core](https://github.com/hobbyquaker/mqtt-interfaces-core) 0.1
and lgtv2mqtt 3.0 implement. Sections marked **reserved** describe things the
2014 convention or the roadmap define but no 2.x implementation provides yet._

This is the successor of [Architecture.md](Architecture.md) (V0.5, 2015).
That document stays as the historical reference; where the two disagree, this
one wins for anything that announces spec `2.x`.

_Not associated with or endorsed by http://mqtt.org_

## 1. Scope

mqtt-smarthome is a **convention**, not a protocol: it says how a program that
bridges some hardware or service ("interface", "adapter", `xyz2mqtt`) presents
itself on an MQTT broker so that logic engines, dashboards, fleet managers and
Home Assistant can use every adapter the same way.

It deliberately does **not** abstract devices into higher-level concepts
("a light", "a cover"). Every adapter exposes what its hardware has, as items;
abstraction, if needed, happens above the bus — or through the optional
interface description (section 8), which maps items to Home Assistant
entity types for consumers that want it.

The words MUST / SHOULD / MAY are used as in RFC 2119.

## 2. Versioning and conformance

- The spec has a version `MAJOR.MINOR`. Implementations state the version
  they implement (libraries in their README, adapters in `<name>/info`, see
  section 6).
- MINOR versions only add; MAJOR versions may change meaning.
- An adapter conforms to 2.0 if it implements sections 3, 4, 5 and 6
  completely. Sections 7–11 are optional or recommendations; what an adapter
  implements of them is visible in `<name>/info`.

## 3. Topic structure

```
<name>/<function>/<item...>
```

- `<name>` — the instance name. Identifies one running adapter instance.
  MUST be configurable (conventionally `--name`/`-n`), so several instances
  of the same adapter can share a broker. Defaults to a short adapter-specific
  word (`lgtv`, `soundbar`, `wiim`, …). MUST NOT contain `/`, `+`, `#`.
- `<function>` — one of `connected`, `status`, `set`, `get`, `info`,
  `maintenance`. Adapters MAY define additional functions; they SHOULD be
  documented in the adapter's README and MUST NOT collide with the reserved
  ones above.
- `<item...>` — one or more levels defined by the adapter (`volume`,
  `play/position`, `wifi/<ssid>/client/<host>`). Item names SHOULD be
  `snake_case`, stable across versions, and identical under `status`, `set`
  and `get`.

Characters from device names that are not valid in topics (`/`, `+`, `#`)
MUST be replaced (recommended: `_`).

### 3.1 `<name>/connected`

Retained. Integer enum as a plain string:

| value | meaning                                                            |
| ----- | ------------------------------------------------------------------ |
| `0`   | adapter not running / not connected to the broker                  |
| `1`   | connected to the broker, hardware not reachable (off, not paired…) |
| `2`   | connected to the broker and to the hardware, fully operational     |

- `0` MUST be set by Last Will and Testament and SHOULD also be published
  explicitly on graceful shutdown.
- `1`/`2` MUST be published on every transition and on (re)connect to the
  broker.
- Adapters that bridge several devices under one `<name>` publish `2` when at
  least the service they bridge is reachable; per-device reachability goes
  into `status/<device>/…` items. (Change from 2014: `connected` is always a
  plain value, never JSON.)

### 3.2 `<name>/status/<item>`

Published by the adapter. State reports: sensor values, actuator feedback,
device info.

- **Retain**: persistent state (volume, temperature, a client list) MUST be
  retained; one-shot events (button press, a voice command, a connect event)
  MUST NOT be retained. Values that change constantly and are worthless when
  stale (play position) SHOULD NOT be retained.
- Published on start/connect (full state), on every change reported by the
  hardware (including changes made via other ways, e.g. the device's own
  remote or app), and again after a broker reconnect (re-publish of all
  retained items so the broker state is complete).
- An adapter SHOULD publish a status only when the value or its timestamp
  meaningfully changed; it MUST NOT republish unchanged state in a tight loop.
- Items that no longer apply (a removed client) SHOULD be cleared by
  publishing an empty retained payload.

### 3.3 `<name>/set/<item>`

Published by consumers to request a change. Same item hierarchy as `status`.
Never retained. The adapter MUST

- accept plain values and `{"val": …}` JSON transparently (section 5.3),
- validate and clamp (ranges, enums) and log rejected requests at `warn`,
- NOT echo the request as a status; the status follows from the hardware's
  feedback (or, if the hardware gives none, from the adapter's own state after
  the action succeeded).

Additional levels below the item MAY carry parameters
(`set/<device>/speak`, `set/light/1/brightness`).

Adapters that accept arbitrary protocol-level commands (lgtv2mqtt `--raw-set`,
wiim2mqtt `--raw-set`) MUST keep that behind an opt-in flag and say so in
`info`.

### 3.4 `<name>/get/<item>` — reserved

From 2014: a write of any payload requests an active read of `<item>`; the
result is published as `status`. Never retained. No 2.x implementation yet;
adapters that support it announce `get: true` in `info`.

### 3.5 `<name>/command` — dropped

The 2014 free-form command channel is not part of 2.x. Use `set/<item>`
with parameter levels or adapter-specific functions instead.

## 4. Retain and QoS

- Retain rules as in 3.1–3.4. Rule of thumb: "would a consumer that starts
  now want to see this?" → retain.
- QoS 0 for everything. QoS 1 is discouraged (duplicates in `set` repeat
  hardware actions, duplicates in `status` repeat events); QoS 2 is
  unnecessary on a local bus.

## 5. Payloads

### 5.1 Plain values

A plain value is the UTF-8 text of a boolean (`true`/`false`), a number
(`12`, `-3.5`) or a string, without quotes. Lists and structured values are
JSON (`["HDMI_1","HDMI_2"]`, `[{"id":"netflix","title":"Netflix"}]`).

### 5.2 JSON status objects

A status MAY instead be a JSON object with at least `val`:

```json
{"val": 17.9, "ts": 1421792677000, "lc": 1421792677000}
```

| field | type   | meaning                                                                   |
| ----- | ------ | ------------------------------------------------------------------------- |
| `val` | any    | the value as it would be published plain (required)                       |
| `ts`  | int    | when the value was obtained, ms since epoch                               |
| `lc`  | int    | when the value last **changed**, ms since epoch (`lc <= ts`)              |

`ts` and `lc` MUST both be present when the object form is used. Adapters MAY
add interface-specific properties; they MUST be grouped in one sub-object
keyed by the adapter's mnemonic, never as additional top-level fields, so
consumers can tell them from future spec fields:

```json
{"val": true, "ts": 1421792677000, "lc": 1421792677000,
 "unifi": {"ap_mac": "aa:bb:cc:dd:ee:ff", "rssi": -61}}
```

(This mirrors node-red-contrib-ccu, which publishes `{val, ts, lc, hm: {…}}`.)

An adapter MUST use one form consistently for all `status` items of an
instance (consumers must not have to guess per item). Which form is active is
selectable per instance (section 9) and announced in `info.jsonPayloads`.

The JSON object form is the **default**. Adapters SHOULD publish `{val, ts, lc}`
unless configured otherwise: `ts`/`lc` are the one thing a consumer cannot
reconstruct from a retained plain value (age of a reading, "open since"),
and the interface description (section 8) hides the difference from Home
Assistant. The plain form is the opt-out for consumers that cannot parse
JSON.

### 5.3 `set` payloads

Consumers MAY send a plain value or `{"val": …}`; adapters MUST accept both
and unwrap `val`. Other JSON objects are passed to the item handler as
structured parameters (e.g. `{"id": "…", "params": {…}}` for an app launch).

Conversions adapters MUST apply:

- booleans: `true`/`false`, `1`/`0`, `on`/`off`, `yes`/`no`, case-insensitive
- numbers: decimal text; rounded/clamped to the item's range
- enums: case-insensitive name match; numeric index MAY be accepted

## 6. Introspection: `<name>/info`

Retained JSON describing the running instance, published on every broker
connect, so tools can find all adapters with `+/info`:

| field          | type   | example / meaning                                      |
| -------------- | ------ | ------------------------------------------------------ |
| `name`         | string | adapter package name, `lgtv2mqtt`                      |
| `version`      | string | adapter version, `3.0.0`                               |
| `spec`         | string | spec version implemented, `2.0`                        |
| `node`         | string | runtime version (`node`; other runtimes use their key) |
| `host`         | string | hostname                                               |
| `pid`          | int    |                                                        |
| `started`      | string | ISO 8601 start time (uptime is derivable)              |
| `maintenance`  | bool   | maintenance topics enabled (section 7)                 |
| `jsonPayloads` | bool   | status form in use (5.2) — _to add_                    |
| `get`          | bool   | supports `get/` (3.4) — _to add, default false_        |

plus adapter-specific extras, grouped in a sub-object keyed by the adapter's
mnemonic as in 5.2 (`"tv": {"address": …}`). Cleared (empty retained
payload) on graceful shutdown is NOT required; a consumer combines `info` with
`connected` to see liveness.

## 7. Maintenance topics

Optional but on by default in the reference implementation; MUST be
disableable (`--no-maintenance`) and the state is in `info.maintenance`.

| topic                             | payload                              | effect                                                                   |
| --------------------------------- | ------------------------------------ | ------------------------------------------------------------------------ |
| `<name>/maintenance/set/loglevel` | `error` \| `warn` \| `info` \| `debug` | change the log level at runtime (not persisted)                          |
| `<name>/maintenance/set/restart`  | any                                  | graceful shutdown (`connected` → `0`) and exit 0; the supervisor restarts |

Never retained. Unknown maintenance topics/commands are logged at `warn` and
ignored. Because these topics let anyone on the bus restart an adapter,
brokers SHOULD use authentication and ACLs; the reference `--install` keeps
broker credentials in a root-only env file.

## 8. Interface description (Home Assistant MQTT discovery)

2.x adopts **Home Assistant's MQTT discovery, device-based format** as the
way an adapter describes its items as entities. It is the only widely
deployed, documented entity-description format; consumers other than HA
(dashboards, fleet managers) are expected to read the same payloads.

- Topic: `<ha-prefix>/device/<id>/config`, retained; `<ha-prefix>` defaults
  to `homeassistant`; `<id>` is `<adapter>_<name>` (plus a device id where
  one `<name>` bridges several devices).
- On by default; `--no-ha-discovery` disables it **and clears** a previously
  retained config.
- Availability MUST point at `<name>/connected` with
  `payload_available: "2"` (an adapter MAY use `1` for entities that work
  without the device, e.g. a wake-on-LAN power switch).
- `state_topic` = `<name>/status/<item>`, `command_topic` =
  `<name>/set/<item>`; with JSON status (5.2) the entity carries
  `value_template: "{{ value_json.val }}"`.
- The discovery payload SHOULD include `origin` (`name`, `sw`, `url`) and is
  republished whenever the entity set changes (e.g. after `model` or an input
  list is learned) and on every broker connect.
- Payload format is pinned to HA's format as of 2025; validation against HA's
  schema in CI is an implementation concern (master roadmap B-8).

## 9. CLI and environment conventions (recommended)

Not required by the convention but implemented by the reference library and
assumed by the fleet manager. Option names are long-form kebab-case; every
option is also readable from the environment as `<ADAPTER>_<OPTION>`
(`LGTV2MQTT_MQTT_URL`), precedence CLI > env > default. The broker options
additionally fall back to unprefixed `MQTT_URL`, `MQTT_USERNAME`,
`MQTT_PASSWORD`, `MQTT_CLIENT_ID_PREFIX`, `MQTT_TLS_CA` so one file can
configure every adapter on a host.

| option                           | default          | meaning                                               |
| -------------------------------- | ---------------- | ----------------------------------------------------- |
| `-u`, `--mqtt-url` (`--url`)     | `mqtt://localhost` | broker URL incl. `mqtts://`, user:pass allowed      |
| `--mqtt-username/--mqtt-password`|                  |                                                       |
| `--mqtt-client-id-prefix`        |                  | client id = `<prefix><name>_<random>`                 |
| `--mqtt-tls-ca`                  |                  | CA file for `mqtts://`                                |
| `-n`, `--name`                   | adapter-specific | instance name / topic prefix (section 3)              |
| `--json-payloads` / `--no-…`     | on               | `{val, ts, lc}` status objects (5.2); off = plain     |
| `--ha-discovery` / `--no-…`      | on               | section 8                                             |
| `--ha-prefix`                    | `homeassistant`  |                                                       |
| `--maintenance` / `--no-…`       | on               | section 7                                             |
| `-v`, `--verbosity`              | `info`           | `error` \| `warn` \| `info` \| `debug`                |
| `--install` / `--uninstall`      |                  | systemd template unit `<adapter>@<name>` (needs root) |
| `--config-schema`                |                  | print a JSON Schema of all options and exit           |
| `--raw-set`, `--publish-raw`     | off              | protocol-level escape hatches, adapter-specific       |

Device address options are adapter-specific (`--tv`, `--address`, …).
`--address auto` / `--discover` are **reserved** for device discovery
(section 10).

Deviations in existing adapters that 2.x ports should fix:
`--topic-prefix` → `--name`, `--verbose` → `--verbosity`,
single-letter-only options → long names with short aliases.

## 10. Device discovery hints — reserved

An adapter MAY declare how its devices can be found on the LAN (mDNS service
type, SSDP search target, TCP port probe, MAC OUI) plus a template for the
resulting instance config. The reference library will provide `--discover
[--json] [--timeout]` (one-shot scan, print candidates) and `--address auto`
(start only on exactly one match). Format to be defined (master roadmap B-2).

## 11. Logging (recommended)

- Levels `error` > `warn` > `info` > `debug`. `error` is for things a human
  must fix (bad config, missing pairing key); an unreachable device is `warn`
  when it first happens and `debug` while it persists (dedupe repeated
  connection errors); attempts at `debug`, outcomes at `warn`.
- Never swallow an error from the device library; every rejected `set` is
  logged with topic and payload.
- Raw traffic at `debug` with direction prefixes: `mqtt >` (published),
  `mqtt <` (received), `<device> >` / `<device> <`.
- Under systemd (journald detected via `JOURNAL_STREAM`) log lines carry
  syslog priority prefixes `<N>` and no own timestamp, so
  `journalctl -p warning -u <adapter>@<name>` works.

## 12. Migration from the 2014 convention (V0.5)

| 2014                                           | 2.0                                                               |
| ---------------------------------------------- | ----------------------------------------------------------------- |
| `connected` may be JSON                        | always plain `0`/`1`/`2`                                          |
| `status` plain or JSON, adapter's choice       | both valid; JSON is the default, per-instance switch, in `info`   |
| `set` always plain                             | plain or `{"val": …}` or structured JSON                          |
| `get/`                                         | reserved, unchanged semantics                                     |
| `command`                                      | dropped                                                           |
| —                                              | `info`, `maintenance/`, HA discovery, CLI/env conventions         |
| topic prefix configurable (should)             | must; `--name`                                                    |
| item names free-form (spaces, device names)    | `snake_case` recommended; invalid chars replaced                  |

Adapters moving to 2.x do a hard break with a major version bump; no
legacy-topic compatibility flags (master roadmap D-2).

## 13. History

- V0.1–V0.5, 2015 — owagner, see [Architecture.md](Architecture.md).
- 2.0 draft, 2026-08-22 — written from mqtt-interfaces-core 0.1 / lgtv2mqtt
  3.0; adds `info`, maintenance topics, HA discovery as interface
  description, CLI/env and logging conventions; drops `command`; pins
  `connected` to plain values.
