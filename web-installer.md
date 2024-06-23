---
layout: framework
title: Web Installer
---
<script
  type="module"
  src="https://unpkg.com/esp-web-tools@10/dist/web/install-button.js?module"
></script>

# Web Installer
Welcome to the GDoor Web Installer, used to program the latest GDoor firmware
on your GDoor hardware adapter via USB connection.

Please read the [documentation](/documentation/getting-started.html) first
for hardware requirements and how to proceed after flashing it.

<esp-web-install-button
  manifest="assets/firmware/manifest.json"
>
<button class="button" slot="activate">Start Firmware Installation</button>
<span id="unsupported" slot="unsupported">Error: Your browser does not support WebUSB. Please use either Chrome or Edge Browser.</span>
<span id="not-allowed" slot="not-allowed">Error: The Web Installer was not granted permission to access USB.</span>
</esp-web-install-button>


<br><br>

## Firmware Updates

For updating the firmware of your running GDoor adapter, please use the update functionality provided by the GDoor adapter web interface:

1. open [GDoor adapter web interface](http://GDoor/) (in case the link doesn't work, please lookup adapter IP address in your router)
2. click "Update"
3. select `firmware.bin` which you can download from latest [GitHub releases](https://github.com/gdoor-org/gdoor/releases)

Alternatively, you could use the Web Installer again but this would reset all your settings like WiFi and MQTT.