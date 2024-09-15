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
1. Solder the ESP32mini pin headers into the ESP32 mini.
Best is to solder all connections (full headers). If you want to save time,
the pins used by GDoor are marked on the back side of the PCB.
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
5. Once connected, either a captive portal appears or if not, surf to [192.168.4.1](http://192.168.4.1)
to setup the connection to your home WiFi and all needed MQTT settings like:
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

In principle the bus adapter should not pose any risk, yet there are caveats as in reality it is hard to catch all possible variants of bus installations
and houses:

- The Gira bus itself is short circuit proof.

For the GDoor adapter:
- The GDoor bus adapter uses a diode bridge, so  bus polarity does not matter.
- The interfaces to the bus are AC coupled, so no risk to send a wrong voltage level to the bus.
- Yet it has a certain RX and TX capacitance, by principle, as any other Gira device. 
 These should be negligible.
- If your ESP would go wild an send all the time, it could theoretically disturb the bus for all, but not damage it.

There is one connection which may be tricky: The power supply of the GDoor bus adapter:
The bus adapter connects the Ground of the power supply to the ground of the bus.
This should pose no problem ("connect 0V to 0V"), but as a developer we can not know how your Gira installation is done.
Even Gira itself warns about this, as this also may happen in regular installations with their own devices.
e.g. https://partner.gira.de/service/faq/antwort.html?id=1254
Violating this should not impose any damage but may disturb the bus and render it unstable.

The safest method to avoid all of above points, without detailed knowledge, is to:
1. Use a isolating 5V supply e.g. MeanWell HDR-15-5, or a USB power supply without earth connection (most of them).
Do not use a computer or similar to power the GDoor adapter.
2. Do not connect to the USB with your computer while the GDoor adapter is connected to the Gira bus.
Only use Wifi.

Please remember this is an open source project with no guarantee. If in doubt you may want to contact a friend/colleague or similar with electrical or electronic background and ask for help on your specific installation.
