---
layout: framework
title: Web Installer - Alpha Version
---
<script
  type="module"
  src="https://unpkg.com/esp-web-tools@10/dist/web/install-button.js?module"
></script>

# Web Installer - Alpha Version
Welcome to the GDoor Web Installer, used to program the latest GDoor firmware
on your GDoor hardware adapter via USB connection.

Please read the [documentation](/documentation/getting-started.html) first
for hardware requirements and how to proceed after flashing it.

<esp-web-install-button
  manifest="assets/firmware-next/manifest.json"
>
<button class="button" slot="activate">Start Firmware Installation</button>
<span id="unsupported" slot="unsupported">Error: Your browser does not support WebUSB. Please use either Chrome or Edge Browser.</span>
<span id="not-allowed" slot="not-allowed">Error: The Web Installer was not granted permission to access USB.</span>
</esp-web-install-button>

