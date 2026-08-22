# Roadmap

mqtt-smarthome is the _convention_: topic structure, `connected` semantics,
payload rules, and — in its next revision — the interface description every
adapter publishes. This repo holds the convention and an overview of the
ecosystem; it is deliberately vendor-neutral. Implementations (adapters,
libraries, management tools, logic engines, UIs) live elsewhere, whoever
writes them.

The author's reference stack and its detailed plan live in
[mqtt-interfaces/ROADMAP.md](https://github.com/hobbyquaker/mqtt-interfaces/blob/main/ROADMAP.md)
(references like D-8 or B-7 below point there).

## 1. README restructure (B-7)

- [ ] Keep `Architecture.md`, `Software.md`, `Devices.md` and `howtos/`
      untouched — they are linked in the wild.
- [ ] Rewrite `README.md` as the umbrella overview:
  1. **The convention** — one-paragraph summary, link to `SPEC.md` (once it
     exists) and to the original 2014 `Architecture.md`.
  2. **Interfaces** — what an `xyz2mqtt` is; the known-software list
     (`Software.md`, own and third-party); libraries for writing new ones
     (`mqtt-interfaces-core` for Node.js, others welcome).
  3. **Management** — fleet managers; `mqtt-interfaces` (install/update/
     configure, web UI, local and remote hosts).
  4. **Logic & UI** — she (rules engine), feezal (dashboards).
     Sections 2–4 label the author's projects as one reference stack;
     anything MQTT-capable works.
  5. **History** — the 2014 concept and what changed since (hard break per
     adapter, Home Assistant discovery, `info`/maintenance topics, …).
- [ ] Step 1 can ship before the spec: short README + "what's next"
      pointing at the master roadmap.

## 2. SPEC.md — mqtt-smarthome spec 2.x

Successor of `Architecture.md`, written in Phase 1 of the master roadmap.
Libraries and adapters state the spec version they implement. Contents:

- [ ] **Topics** — `<name>/connected` (0/1/2, LWT), `<name>/status/<item>`,
      `<name>/set/<item>`, `<name>/get/<item>`; retain rules (persistent
      state retained, one-shot events not); QoS 0 recommendation.
- [ ] **Payloads** — plain values by default, `{val, ts, lc}` JSON as an
      option; `set` accepts both transparently.
- [ ] **Introspection** — `<name>/info` retained JSON (adapter, version,
      runtime, uptime, host) so tools can discover instances via `+/info`.
- [ ] **Maintenance topics** — `<name>/maintenance/set/loglevel`,
      `<name>/maintenance/set/restart`; must be disableable; security
      implications and broker auth/ACL guidance.
- [ ] **Interface description** — Home Assistant MQTT discovery, device-based
      (`homeassistant/device/<id>/config`), adopted as the entity-description
      format; availability via `<name>/connected` with
      `payload_available: "2"`; pinned format version; how consumers other
      than HA (dashboards, managers) are expected to use it.
- [ ] **Device discovery hints** — declarative per-adapter hint format
      (SSDP/mDNS/TCP/OUI + instance template) and the `--discover` /
      `--address auto` semantics.
- [ ] **CLI/env conventions** (recommended, not required by the convention):
      canonical option names, `<ADAPTER>_*` env vars, shared broker variables
      `MQTT_URL`/`MQTT_USERNAME`/… as fallback.
- [ ] **Logging** — severity semantics and the rules distilled from adapter
      support history (never swallow device errors, dedupe connection
      errors, action-required at `warn` naming the file/setting, raw traffic
      at `debug` with direction prefixes).
- [ ] **Migration notes** 2014 → 2.x.

## 3. Lists

- [ ] Refresh `Software.md`: mark unmaintained entries, add new ones, link
      the 2.x spec version each implements once that exists.
- [ ] `Devices.md`: keep as is, review for dead links.
