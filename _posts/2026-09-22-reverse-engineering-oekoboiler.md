---
title: "Reverse-Engineering My Ökoboiler's Internal UART Bus"
date: 2026-09-22
tags: [Home Automation, Reverse Engineering]
---

My [Ökoboiler](https://www.oekoboiler.com/) heat-pump water heater is an older
generation with zero connectivity — no WiFi, no app, no API. Inside, two circuit
boards, a display/operation panel on the front and the main control board, talk to
each other over a single-wire serial bus. That bus turned out to be the only way
in, so I tapped it with a €5 ESP32, captured the traffic, and reverse-engineered
the protocol. The full write-up, code and captures are on
[GitHub](https://github.com/splattner/oekoboiler-uart-reverse-engineering); this
is the short version.

The result is a local, cloud-free sensor reading water tank temperature (to about
0.3°C — finer than the front panel's own 1°C display), operating mode
(heating/idle/defrost), and PV/solar state — fed straight into ESPHome and Home
Assistant.

## The hardware

The main control board is an ATmega16-based SY-384. It talks to the front panel
over a single-wire, half-duplex UART bus at 2400 baud, 8N1, optically isolated.
The panel is wired with only three conductors: UART data, 5V and GND — no sensors
of its own. That detail mattered: it proved that everything the panel displays,
including the tank temperature, has to arrive over this one wire. It was in there
somewhere.

## Finding structure in the bytes

Dumping the raw stream, a repeating 32-byte frame jumped out, always starting with
the same two header bytes. Byte 5 changes every frame, and XOR-ing bytes 2–29
against it produces a stable, readable payload — after which a constant signature
appears at bytes 6–9, a reliable sanity check that decoding worked. Bytes 30–31
turned out to be a CRC-16/Modbus over the rest of the frame, which let me discard
corrupted frames with confidence. Decoded byte 11 splits the traffic into two
families, one per direction of the conversation.

Two fields fell quickly just by watching decoded byte 10 while poking the
machine: the low bits give operating mode (heating/idle/defrost), and one bit
flags PV/solar mode.

## The PV mode that wasn't supposed to be there

My unit was sold as the cheaper model, advertised *without* PV/solar mode. Except
it basically has one: PV mode is documented in the manual that shipped with the
unit, with its own target temperatures, and the schematic has a terminal labelled
"PV Input" sitting right there on the board. The firmware never lost the feature —
the pricier model just adds something to drive the trigger.

So I enabled PV mode in the settings menu, wired a Shelly 1 Mini Gen3 relay onto
that PV Input terminal, and let Solar Manager close it whenever my system has
surplus solar. The boiler now jumps to its PV setpoint on sunny afternoons, and
the bus decode confirms it by reading the resulting PV state straight back.

## The water-temperature hunt

This was the hard part, and it took days. The obvious approach — note the display
reading, find a matching byte, repeat — kept failing, because the board reads
several temperature sensors (tank, evaporator, ambient, exhaust) and their bytes
all shift with operating state. Two moments showing the same tank temperature on
the display can have very different readings everywhere else on the bus. Spot
checks confound temperature with machine state; you need a real sweep of labelled
data to untangle it.

I re-enabled a camera-and-OCR rig I already had pointed at the front panel, logging
the displayed temperature into InfluxDB alongside every decoded byte, and started
correlating properly — Pearson correlation between the ground-truth display
reading and each of roughly 60 candidate bytes.

The first attempt was a trap: overnight, as the tank cools passively, *every*
byte that drifts downward correlates with temperature at ~0.98, because they're
all cooling together. I needed a heating event — a moment where the tank diverges
from everything else — so I waited for a full day-night "V": cool down overnight,
reheat during the day. Even then, several coil-sensor bytes still scored ~0.9 on
plain correlation, so I ran a time-lag scan — sliding each candidate byte forward
and backward in time before recomputing the correlation. The real tank reading
peaks sharply at lag zero; a coil sensor that merely follows the same daily
heating rhythm peaks 90–120 minutes off, revealing it as a coincidence of timing
rather than the thing itself.

That scan pointed straight at decoded byte 14: it peaks at lag zero, uses the full
0–255 range, and had been dismissed early on as noise precisely because it's
higher-resolution than the display and wraps around. `byte14 mod 64`, plotted
against temperature, is a clean line — falling about 10 units per °C and wrapping
every 6.4°C. A second byte, 15, supplies the coarse band needed to disambiguate
which wrap you're in.

## The payoff

No cloud, no API reverse-engineered from an app — just a wire tap, a lot of
correlation, and a sensor that now feeds temperature, mode and PV state directly
into ESPHome and Home Assistant. The full protocol notes, capture data and code
are in the [repo](https://github.com/splattner/oekoboiler-uart-reverse-engineering)
if you're staring at a similarly silent appliance.
