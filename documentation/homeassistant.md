---
layout: documentation
title: Home Assistant
sidebar:
  - article-menu
---

# Integrate your Gira door system

The following allows you to e.g.
- open the door via Home Assistant
- trigger notifications on mobile devices when a door bell button is pressed
- use any actions sent on the Gira bus to trigger automations

# Connect Home Assistant to GDoor

There are two possibilities:

### MQTT (recommended)

GDoor supports Home Assistant MQTT Auto-Discovery.

1. install [Home Assistant Add-on: Mosquitto broker](https://github.com/home-assistant/addons/blob/master/mosquitto/DOCS.md) and MQTT integration
2. create user to be used for MQTT
3. open [GDoor adapter web interface](http://GDoor/) (in case the link doesn't work, please lookup adapter IP address in your router)
4. set MQTT host to the IP of your Home Assistant installation as well as the created MQTT user name and password
5. now the `GDoor Adapter` device should appear in Home Assistant
6. press the door bell
7. press the "open door" button on your indoor station
8. open the Home Assitant Logbook view and search for "GDoor", now you should see state changes like
   ```
   GDoor changed to {"action": "BUTTON_RING", "parameters": "0360", "source": "A286FD", "destination": "000000", "type": "OUTDOOR", "busdata": "011011A286FD0360A04A"}
   ```
9. look for the `BUTTON_RING` action: note the value of the `parameters` field, this is the unique value of your door bell which you can use to identify your door bell in automations.
10. look for the `DOOR_OPEN` action: note the `busdata` field, if you want to open the door via Home Assistant you need to send this exact value to the MQTT topic `gdoor/bus_tx`



### USB / Serial connection

1. connect the Gdoor adapter to a USB port of the machine where Home Assistant is running
2. edit `configuration.yaml` (e.g. via the [Home Assistant Add-on: Studio Code Server](https://github.com/hassio-addons/addon-vscode)) and add

    ```
    sensor:
      - platform: serial
        name: "GDoor"
        serial_port: /dev/ttyUSB0 # change device if needed
        baudrate: 115200
    ```
   If you're unsure under which device path the serial port got registered, go to Settings -> Logs and choose "Supervisor". Look for `[supervisor.hardware.monitor] Detecting add hardware /dev/ttyUSB`...
3. restart Home Assistant
4. press the door bell
5. press the "open door" button on your indoor station
6. open Home Assitant Logbook and search for "GDoor", now you should see state changes like
   ```
   GDoor changed to {"action": "BUTTON_RING", "parameters": "0360", "source": "A286FD", "destination": "000000", "type": "OUTDOOR", "busdata": "011011A286FD0360A04A"}
   ```
7. look for the `BUTTON_RING` action: note the value of the `parameters` field, this is the unique value of your door bell which you can use to identify your door bell in automations.
8. look for the `DOOR_OPEN` action: copy the value of `busdata` and add the following shell command in `configuration.yaml` to be able to open the door programmatically. Replace `<busdata>` with the copied value.
    ```
    shell_command:
      gdoor_open_door: echo -e '<busdata>' > /dev/ttyUSB0 # change busdata and device if needed
    ```
9. restart Home Assistant



# Examples for typical use cases

### Open door via Home Assistant dashboard / automation

Simply perform the HA action "MQTT: Publish" to replay the busdata which is sent when you open the door via your indoor station, e.g. in a button card:

```
type: button
name: Haustür öffnen
icon: mdi:door-open
show_name: true
show_icon: true
tap_action:
  action: perform-action
  target: {}
  perform_action: mqtt.publish
  data:
    qos: "0"
    payload: 020031ED3F4B0060A1A254791A # check in Logbook which busdata is sent when you open the door bell via your indoor station
    topic: gdoor/bus_tx
```

When using the USB / Serial connection, replace `tap_action` to call the above defined service `shell_command.gdoor_open_door`:

```
tap_action:
  action: call-service
  service: shell_command.gdoor_open_door
```

### Send notification to mobile devices on door/floor bell

Example automation:

```
alias: Türklingel
triggers:
  - trigger: mqtt
    topic: gdoor/bus_rx
action:
  - choose:
      - conditions:
          - condition: template
            value_template: >-
              {{ trigger.payload_json.action == 'BUTTON_RING' and
              trigger.payload_json.parameters == '0360' }} # check in Logbook which parameter matches your door bell
        sequence:
          - service: notify.all_smartphones # adjust to your notification group/device
            data:
              data:
                push:
                  interruption-level: time-sensitive
              message: Türklingel (außen)
      - conditions:
          - condition: template
            value_template: >-
              {{ trigger.payload_json.action == 'BUTTON_FLOOR' and
              trigger.payload_json.parameters == 'FF6F' }} # check in Logbook which parameter matches your floor bell
        sequence:
          - service: notify.all_smartphones # adjust to your notification group/device
            data:
              data:
                push:
                  interruption-level: time-sensitive
              message: Türklingel (innen)
mode: single
```


For USB / serial connection, adjust `triggers` to

```
triggers:
  - platform: state
    entity_id:
      - sensor.gdoor
```

and `conditions` to e.g.:

```
      - conditions:
          - condition: state
            entity_id: sensor.gdoor
            attribute: action
            state: BUTTON_RING
          - condition: state
            entity_id: sensor.gdoor
            attribute: parameters
            state: "0360" # check in Logbook which parameter matches your door bell
```