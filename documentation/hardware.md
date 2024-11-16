---
layout: documentation
title: Firmware
sidebar:
  - article-menu
---

# Overview
The hardware is currently in version 3.1

- v3.0: First release with weak sending and a hardware bug in the receive path.
- v3.1: Hardware adapter procured and currently being tested.
- v3.1-1: This is a minor release with a changed screw terminal, due to parts availability at JLCPCB.
- v4.0 (development): This is a GDoor version currently under development. Changed:
  - Audio support, GDoor acts as Voip phone
  - Bus voltage measurement

# Design
You can download the PDF version of the schematic [here](/assets/design/esp32.pdf).
The KiCAD source files are in the [Github Repo](https://github.com/gdoor-org/gdoor/tree/main/hardware/esp32).
<div class="image">
<img src="/assets/images/doc-schematics.png"/>
</div>
<div class="image">
<img src="/assets/images/doc-pcb.png"/>
</div>