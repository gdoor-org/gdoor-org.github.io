---
layout: default
---
<script
  type="module"
  src="https://unpkg.com/esp-web-tools@10/dist/web/install-button.js?module"
></script>

# Web Installer
Welcome to the GDOOR Web Installer, used to program the latest GDOOR firmware
on your GDOOR hardware adapter. Please read the [documentation](./documentation.html) first
for hardware requirements needed before flashing the firmware.

<esp-web-install-button
  manifest="https://raw.githubusercontent.com/gdoor-org/gdoor/main/firmware/esp32/gdoor/manifest.json"
>
<button id="installer" slot="activate">Start Firmware Installation</button>
<span id="unsupported" slot="unsupported">Error: Your browser does not support WebUSB. Please use either Chrome or Edge Browser.</span>
<span id="not-allowed" slot="not-allowed">Error: The Web Installer was not granted permission to access USB.</span>
</esp-web-install-button>