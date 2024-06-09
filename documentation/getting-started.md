---
layout: documentation
title: Documentation
sidebar:
  - article-menu
---
<div class="image">
<img src="/assets/images/doc-schematics.png"/>
<img src="/assets/images/doc-pcb.png"/>
</div>

# Getting Started
{: #started}
You need the GDoor hardware adapter + ESP32, see more details under [Buy](./buy.html).

The adapter was successfully tested with
- Gira Türkommunikations-System Steuergerät Audio (1287 00)
- Gira Türkommunikations-System Steuergerät Video (1288 00)
- Gira Wohnungsstation AP (1250 015)
- Gira Wohnungsstation Video AP Plus (1239 03)
- Gira Türstation AP 1-fach (1266 65/66/67)
- Gira Türstation AP 3-fach (1267 65/66/67)

# Building the Adapter
After you procured an GDoor adapter ([Buy](/buy.html)) you need to:
1. Solder the ESP32mini pin headers into the ESP32 mini
2. Plug the ESP32mini into the GDoor adapter.

> #### Warning:
>
> Be aware of the orientation! The USB port of the ESP32mini needs to point
> to the side with the bus connector.
{: .block-tip }

<p align="center">
<a href="/assets/images/hw3.1/resize-DSC_1440.jpg" target="blank"><img src="/assets/images/hw3.1/thumb-DSC_1440.jpg" width="200px"/></a>
</p>

# Initial Setup
{: #setup }
With the fully build GDoor hardware you can get started to connect it to the bus:

1. Connect the adapter via USB to your PC.
2. Flash the GDoor adapter firmware via our [Web Installer](./web-installer.html)
3. After the firmware is flashed, the adapter will create a WiFi called `GDoor`.
4. Connect to it with the password `12345678`
5. Once connected, you can surf to [192.168.4.1](http://192.168.4.1)
and setup the connection to your home WiFi and all needed MQTT settings like:
- MQTT Broker Network Address
- MQTT Username & Password (both optional)
- MQTT topic where door bus data is send to
- MQTT topic, to which the adapter subscribes and sends received data onto the bus
- RX Pin: Leave it default, useful if you still have an old v3.0 adapter.
- RX Sensitivity: If you do not receive all messages you can increase the sensitivity,
  if you receive false bus message you can decrease it.
- Debug: If you enable debug mode, more output will be send via the USB serial port.
  Also bus messages which do not pass the checksum validation, will be published via MQTT.
6. After successfully setting up the adapter, you can connect it to the Gira TKS Bus.
The bus polarity does not matter as the bus adapter automatically connects the right way.
7. You can disconnect your computer via USB, but the adapter needs to be powered by *either* the USB bus,
or the dedicated and labeled pins with a good 5V, min. 1A power supply.

# Connection Warning
{: #warning }
<p align="center">
<img src="/assets/images/doc-pinout.png" height="200px"/>
</p>

Be careful if you connect an external 5V supply via the dedicated pins,
ensure correct polarity and voltage levels!
Do not connect the bus to this input!
