---
layout: documentation
title: Audio / VOIP
sidebar:
  - article-menu
---

# Requirements
A GDoor adapter with at least version 4.0 is needed.

# Setup
Go to the configuration page of your GDoor device.
You can setup the connection string of you VOIP
server.
The scheme is `udp://username:password@server:portnumber`

# Configuration
GDoor currently offers 4 slots.
Each slot is a individual configuration to handle a audio related action
and can be enabled or disabled independently.

A slot has the following settings:
- Outgoing Call: Enable or disable that this slot initiates outgoing VOIP calls.
- Trigger Action: The basic action which trigger this slot, it is the `ACTION` field in the incoming bus message which needs to match.
  - BUTTON_RING: An action is triggered, when a ring button on a outdoor station is pressed.
  - CALL_INTERNAL: An action is triggered, when a internal call is initiated.
- Outdoor Address: This is a 2-Byte hexadecimal value. Do not prefix `0x`.
This address is used to match the source field of the incoming bus message,
so that the slot is only triggered for a certain station. Leave empty to trigger an all stations.
- Button No.: The button number to trigger. It is the first byte of the parameter field in the incoming bus message.
Leave empty to trigger an all button presses.
- Dest. Phone No.: The phone number which is called by GDoor when this slot is triggered.

# Tested Setups
Currently tested are:
- FritzBox as VOIP server
- GDoor v4.0 as software VOIP client.
- Linphone as software VOIP client, which GDoor calls, when a button is pressed