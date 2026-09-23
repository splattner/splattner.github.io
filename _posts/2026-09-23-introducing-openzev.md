---
title: "Introducing OpenZEV"
date: 2026-09-23
tags: [Home Automation, Open Source]
---

If you share one grid connection with your neighbours — a "ZEV" (Zusammenschluss
zum Eigenverbrauch) or "vZEV" in Swiss terminology — someone has to work out who
owes what for the solar that was produced and consumed behind that single meter.
That someone was me, doing it by hand in a spreadsheet, for my own ZEV. It got old
fast. So I built [OpenZEV](https://github.com/splattner/openzev), an open source
billing platform for exactly this problem, and put it out at
[openzev.ch](https://www.openzev.ch/) in case it's useful for other communities too.

## What a ZEV billing problem actually looks like

A ZEV pools solar production and consumption across several households behind one
grid connection. Correct billing means importing every participant's meter data,
splitting each 15-minute interval of shared production and consumption fairly
between them, pricing it against the community's tariffs, and turning that into an
invoice someone can actually pay. Do it manually and it's tedious and error-prone;
get it wrong and someone is quietly overpaying for power they didn't use.

OpenZEV does that whole chain end to end.

## What it does

- **Communities, roles and metering points.** Four roles — admin, ZEV owner,
  participant, guest — each see the slice of data relevant to them. Assignments
  between participants and metering points carry validity dates, so someone moving
  out mid-month is billed correctly and the next tenant picks up cleanly from where
  they left off.
- **Metering imports.** CSV/Excel with configurable column mapping, or SDAT-CH for
  utilities that speak it. Every import runs as a preview first, with a per-row
  protocol, so you see exactly what a file will do before it touches the database.
  A data-quality view flags gaps, duplicates and implausible readings.
- **Tariffs and billing.** Allocation runs per timestamp — every 15-minute
  interval is split across participants and priced with whichever tariff version
  was valid at that exact moment. Tariffs are versioned, with high/low bands and
  seasonal windows, so an invoice raised last year still prices at last year's
  rate even if tariffs have changed since. Dynamic price series and a grid
  operator's machine-readable tariff file (Art. 7b StromVV) are both supported.
- **Documents.** Invoices render as PDF/A-3b with a Swiss QR-bill payment slip.
  Annual statements and participation contracts come out of the same renderer,
  and every issued version is preserved exactly as it was sent.
- **Feasibility planning.** Before you've even founded a community, OpenZEV can
  estimate savings, payback, ROI and NPV from modelled producers and consumers —
  useful for figuring out whether starting a ZEV is worth it in the first place.

It's self-hosted and licensed AGPL-3.0.

## How it was built

OpenZEV was built with generous AI assistance — right down to the specs, ADRs and
user docs. That's a deliberate trade-off, not an oversight: some design choices
might look a little unconventional compared to what a more seasoned team would
land on. The project is optimized for learning, experimentation, and running my
own ZEV well, not for enterprise-grade process perfection. It's shipped as-is —
check your own data and billing outputs before they reach a participant.

If you're running (or thinking about starting) a Swiss ZEV or vZEV, the code is on
[GitHub](https://github.com/splattner/openzev) and the project has its own home at
[openzev.ch](https://www.openzev.ch/).
