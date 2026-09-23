---
title: "Bridging digitalSTROM to the Rest of My Smart Home"
date: 2026-09-20
tags: [Home Automation]
---

[digitalSTROM](https://www.digitalstrom.com/) is a Swiss home automation system
built around smart relays wired directly into the electrical installation — solid
for switching and metering, but an island on its own unless something bridges it
to everything else. That bridge is a "vDC" (Virtual Device Connector), a protocol
digitalSTROM's own server uses to talk to third-party devices as if they were
native digitalSTROM hardware. I wanted my digitalSTROM installation to see my Home
Assistant, WLED, Tasmota and Zigbee2MQTT devices, so I wrote one:
[digitalstrom-vdc-bridge](https://github.com/splattner/digitalstrom-vdc-bridge),
a Go implementation of the vDC API with a web-based configuration UI and a plugin
system for bridging in other systems.

## What it does

- **Full vDC API compatibility** — the protobuf wire protocol, all 16 inbound
  message types, scenes, channels, sensor descriptors, and DNS-SD advertisement
  so the digitalSTROM Server discovers the bridge automatically on the network.
- **A web UI**, embedded in the binary with no separate server needed, for
  schema-driven plugin management, device discovery and bridge mapping.
- **A plugin system** built around a shared MQTT broker connection, with plugins
  for Home Assistant (discovers lights and sensors via its WebSocket API), WLED
  (mDNS discovery), Tasmota (via Tasmota's own MQTT discovery), and Zigbee2MQTT
  (via its bridge/devices topic).
- **Persistent storage**, so scenes, device configs and bridge mappings survive a
  restart.

It's heavily inspired by, and builds on protocol work from,
[plan44/vdcd](https://github.com/plan44/vdcd) by Lukas Zeller — the vDC API
protocol handling and device model abstractions follow the patterns established
in that project, and I'm grateful for the open reference implementation.

## Running it

It ships as a container and, more usefully for most of my own setup, as a Home
Assistant add-on. The standalone container needs `--network host`, because mDNS
(DNS-SD) discovery relies on multicast traffic that a Docker bridge network would
otherwise isolate — without it, the digitalSTROM Server simply never finds the
bridge on its own. That does mean the web UI and API are reachable by anything on
the LAN with no login by default, so there's an optional HTTP Basic Auth flag for
the standalone case; running it as a Home Assistant add-on instead gates access
through your HA session via ingress, which is the simpler and safer path if
you're already on Home Assistant.

If you're running digitalSTROM alongside anything else — Home Assistant, WLED,
Tasmota, Zigbee2MQTT — the bridge, its docs and the Home Assistant add-on
repository are on [GitHub](https://github.com/splattner/digitalstrom-vdc-bridge).
