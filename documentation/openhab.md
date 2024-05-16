---
layout: documentation
title: Openhab
sidebar:
  - article-menu
---

# Integrate your Gira door system

The following allows you to e.g.
- open the door via Openhab
- trigger notifications on mobile devices when a door bell button is pressed
- use any actions sent on the Gira bus to trigger automations

# Connect to Gdoor
Use the  [MQTT Binding](https://www.openhab.org/addons/bindings/mqtt.generic/) and the `JSONPath Transformation` to connect OpenHab to GDoor via MQTT.

# Get needed parameters
### Open your door
To open your door, you need to copy what your indoor station is sending,
when the "door open" button is pressed.
Use e.g. [MQTT Explorer](http://mqtt-explorer.com/) to listen to your MQTT broker,
and get the value of `busdata`, when you press the door opener button.

### Door Bell
To detect a door bell ring,
you need the `parameter` value of your door bell.
Use e.g. [MQTT Explorer](http://mqtt-explorer.com/) to listen to your MQTT broker,
and get the value of `parameter`, when you press the door bell button.

# Example
In the following example you need to change the following values:
- `192.168.0.42` (host): IP of your MQTT broker, you may also need to set the auth credentials of your broker.
- `gdoor/bus_tx` (commandTopic): Set it to the value you used during initial configuration ([Getting Started](/documentation/getting-started.html))
- `gdoor/bus_rx` (stateTopic): Set it to the value you used during initial configuration.
- `011011A286FD0360A04A` (formatBeforePublish): Set it to the `busdata` value you received for a door opener action.
- `0360` (on): The `parameter` field of the door bell action.
- `availabilityTopic=homeassistant/sensor/gdoor/data/config/08_B6_1F_35_1E_80`: Is optional, you can find it via the MQTT autodiscovery,
   used to indicate if the adapter is online or offline

For .things file:
```
Bridge mqtt:broker:myInsecureBroker [ host="192.168.0.42", secure=false ]

Thing mqtt:topic:gdoor "GDoor" (mqtt:broker:myInsecureBroker) [availabilityTopic="homeassistant/sensor/gdoor/data/config/08_B6_1F_35_1E_80", payloadAvailable="online", payloadNotAvailable="offline"] {
    Channels:
    Type switch : door_opener "Open Door" [ commandTopic="/gdoor/bus_tx", formatBeforePublish="011011A286FD0360A04A"]
    Type switch : door_bell "Door Bell" [ stateTopic="/gdoor/bus_rx", transformationPattern="JSONPATH:$.parameters", on="0360", off="0000" ]
    Type string : door_bus "Doorbus Message" [ commandTopic="/gdoor/bus_tx",  stateTopic="/gdoor/bus_rx", transformationPattern="JSONPATH:$.busdata"]
    Type string : door_message "Message" [ commandTopic="/gdoor/bus_tx",  stateTopic="/gdoor/bus_rx"]
    Type string : door_action "Action" [ stateTopic="/gdoor/bus_rx", transformationPattern="JSONPATH:$.action"]
}




