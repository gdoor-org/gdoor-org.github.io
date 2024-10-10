---
layout: documentation
title: TKS Protocol
sidebar:
  - article-menu
---

# Hardware Layer
The bus is formed by two wires and is single ended, meaning one of the wire carries
power and data and the other wire is ground. For the GDoor hardware adapter, the polarity doesn't matter.

To power the bus devices, a central device called controller powers the bus with ~26 Vdc.
Bus commands and analog data (audio and video!) are modulated onto this 26V.
The exact electrical scheme for the modulations is not known, but capacitive coupling the signals
on the 26 V seems to work.

Digital bus commands indicate the start and end of analog audio/video transmission,
each participating bus device decides on its own to:
- send audio/video on the bus
- receive audio/video from the bus

The outdoor station can be commanded to send audio/video to the bus, the indoor stations
will send audio only if a user pressed the "call accept" button.

On the bus, the following elements are transmitted and received:
- Digital Signaling: Control Signals like Door Opener, Lights etc (see `Digital Signaling` below).
- Audio: Audio is transferred as analog audio, directly coupled to the bus.
  More details are in the [Audio Github issue](https://github.com/gdoor-org/gdoor/issues/20).
- Video: Video is FM modulated at ~11MHz. It is composite video, PAL, 625, 25 FPS  worked to decode it.
  More details are in the [Video Github issue](https://github.com/gdoor-org/gdoor/issues/21).

# Digital Signaling
![Example of Bus voltage](/assets/images/busvoltage.png)

The digital bus signal is based on a non standardized modulation scheme, similar to OOK, but not quite OOK.
A ~ +-2V, 60 kHz sine carrier is turned on and off, where the duration of the carrier indicates a one or zero bit value. These pulses (aka bits) are separated with a fixed length pause (with no carrier signal).
So it is similar to OOK, but the bit period depends on the bit value.

All data transmission starts with a start bit which contains ~60 sine periods. A one is signaled by ~12 sine periods and a zero is signaled by ~32 sine periods.

# Bits and Bytes
![Bit stream order](/assets/images/wavedrom-bitstream.png)

Data transmission is LSB first and each Byte has an additional 9th odd parity bit.
The last byte in the transmission is kind of a checksum, literally the sum of all previous bytes.

# Bus protocol / Frame
![Byte Frame](/assets/images/wavedrom-byteframe.png)
The exact meaning of the bytes is unknown, but certain clues can be made by observing the bus and sending
to the bus and observing device behavior:

| Byte          | Description   |
| ------------- | ------------- |
| ?Length?      | Maybe frame length. Content is 0x01 if no destination fields in frame, otherwise 0x02  |
| ?Status?      | Unknown, fixed values for different commands  |
| Action        | The real command, like "open door", "accept call" etc.  |
| Source        | 3 Byte value with device bus address, unique per device!  |
| Parameter     | 2 Byte value which e.g. specifies pressed button  |
| Device Type   | Fixed value for each hardware device type  |
| Destination   | 3 Byte value with device bus address, unique per device!  |
| CRC           | Sum of all previous byte (8 bit, without parity bit) values |

### ?Length?

| Byte-Value    | Description   |
| ------------- | ------------- |
| 0x01          | Frame with 9 Bytes, no destination bytes |
| 0x02          | Frame with 12 Bytes, incl. destination bytes |

### ?Status?

| Byte-Value    | Description   |
| ------------- | ------------- |
| 0x00          | Often in combination with ?Length? = 0x02 |
| 0x10          | Often in combination with ?Length? = 0x01 |

### Action

| Byte-Value    | Description   |
| ------------- | ------------- |
| 0x00          | Programming mode - Stop (`CTRL_PROGRAMMING_STOP`)|
| 0x01          | Programming mode - Start (`CTRL_PROGRAMMING_START`)|
| 0x02          | Door opener programming - Stop (`CTRL_DOOROPENER_TRAINING_STOP`)|
| 0x03          | Door opener programming - Start (`CTRL_DOOROPENER_TRAINING_START`)|
| 0x04          | Learn doorbell button (`CTRL_BUTTONS_TRAINING_START`)|
| 0x05          | Confirm learned doorbell button (`CTRL_DOORSTATION_ACK`)|
| 0x08          | Reset device configuration (announcement by device itself) (`CTRL_RESET`)|
| 0x0F          | Confirm learned door opener (`CTRL_DOOROPENER_ACK`)|
| 0x11          | Door bell button pressed - which button is specified in ?Param? (`BUTTON_RING`)|
| 0x12          | Internal call from one to another indoor station - which station is specified in ?Param? (`CALL_INTERNAL`)|
| 0x13          | Floor bell button pressed (`BUTTON_FLOOR`)|
| 0x20          | End audio/video transmission (`AUDIO_VIDEO_END`)|
| 0x21          | Request audio (`AUDIO_REQUEST`)|
| 0x28          | Request video (`VIDEO_REQUEST`)|
| 0x31          | Open door (`DOOR_OPEN`)|
| 0x41          | Light button pressed (`BUTTON_LIGHT`)|
| 0x42          | Generic button pressed - which button is specified in ?Param? (`BUTTON`)|

see [mapping of actions in the firmware source code](https://github.com/gdoor-org/gdoor/blob/main/firmware/esp32/gdoor/src/gdoor_data.cpp).

### Source
3 Byte of device address

### Parameter
2 Bytes with parameters for the Action field.
E.g. a door station with multiple buttons encodes the pressed key number.

#### 1. Action = BUTTON_RING (0x11), BUTTON_LIGHT (0x41), DOOR_OPEN (0x31)
- First Byte: Most likely button number (and maybe attachment number, see
[Issue 33](https://github.com/gdoor-org/gdoor/issues/33)).
- Second Byte: 0x60 for normal press, 0xA0 for long press (indicates something in bit 8 and bit 7).

#### 2. Action = CALL_INTERNAL (0x12)
- First Byte:  Unconfirmed - Most likely button number (and maybe attachment number, see
[Issue 33](https://github.com/gdoor-org/gdoor/issues/33)).
- Second Byte: Unconfirmed - 0x60 for normal press.

#### 3. Action = BUTTON (0x42)
- First Byte: Always bit 7 seems to be set (0x40), lower bits seem to indicate button number (1,2,3...).
- Second Byte: 0x40 for ON, 0x50 for OFF.

#### 4. Action = VIDEO_REQUEST (0x28)
- First Byte:
  - 0x00: Disable Camera, if second byte = 0
  - 0x02: Enable Camera, with outdoor camera light
  - 0x03: Enable Camera, without camera light
- Second Byte: Video Frequency. If frequency is 0 and first byte is 0x01, the Video is turned off.

#### 5. Action = CTRL_DOOROPENER_TRAINING_START (0x03)
- First Byte:
  - 0x00: Start training of door opener
  - 0x01: Send by switching actuator (1289 00), when switched to mode "door opener"
  - 0x02: Send by switching actuator (1289 00), when switched to mode "impulse"
  - 0x04: Send by switching actuator (1289 00), when switched to mode "timer/min"
  - 0x08: Send by switching actuator (1289 00), when switched to mode "timer/sec"
  - 0x10: Send by switching actuator (1289 00), when switched to mode "switch"
- Second Byte: Currently unkown

### Type

| Byte-Value    | Description   |
| ------------- | ------------- |
| 0xA0          | Door station (`OUTDOOR`)|
| 0xA1          | Indoor station (`INDOOR`)|
| 0xA2          | Indoor headset (`INDOOR_RECEIVER`)|
| 0xA3          | Controller (`CONTROLLER`)|
| 0xA4          | TKS switching actuator (`ACTUATOR`)|
| 0xA5          | TKS TK Gateway (Analog phone) (`GATEWAY_TK`)|
| 0xA6          | TKS Chime (`CHIME`)|
| 0xA7          | TKS Button Interface (`BUTTON_IF`)|
| 0xA8          | TKS IP Gateway (`GATEWAY_IP`)|

### Destination
3 Byte of device address

# Bus Messages

### Open Door

| -----|------|------|--------|--------|--------|------|------|------|--------|--------|------- |
| 0x02 | 0x00 | 0x31 | src[0] | src[1] | src[2] | 0x00 | 0x00 | 0xA0 | dst[0] | dst[1] | dst[2] |

The door opener with address `dst` ignores `src` (can be any byte values),
it also ignores the hardware type (0xA0).
The indoor station sets `dst` to the outdoor station if there is an active audio/video transmission. Otherwise it sets the controller as `dst`.

### Ring button

| -----|------|------|----------------|----------------|----------------|--------|------|------|
| 0x01 | 0x10 | 0x11 | doorstation[0] | doorstation[1] | doorstation[2] | button | 0xA0 | 0xA0 |

Doorstation ring button is 0x01, 0x02, 0x03 ...

### Request audio

| -----|------|------|-----------|-----------|-----------|------|------|------|---------|---------|-------- |
| 0x02 | 0x00 | 0x21 | indoor[0] | indoor[1] | indoor[2] | 0x00 | 0x00 | 0xA1 | door[0] | door[1] | door[2] |

Door station ignores indoor byte values and hardware type (0xA1).
As soon as door station receives this command, it sends analog audio onto the bus.

### Request video

| -----|------|------|-----------|-----------|-----------|------|------|------|---------|---------|-------- |
| 0x02 | 0x00 | 0x28 | indoor[0] | indoor[1] | indoor[2] | 0x00 | 0x00 | 0xA1 | door[0] | door[1] | door[2] |

Door station ignores indoor byte values and hardware type (0xA1).
As soon as door station receives this command, it sends analog video onto the bus.

> #### Note: 
> The indoor station also sends this command when viewing a missed call or opening the menu.
> The parameters are different in this case.
{: .block-tip }

### End audio/video transmission

| -----|------|------|-----------|-----------|-----------|------|------|------|---------|---------|-------- |
| 0x02 | 0x00 | 0x20 | indoor[0] | indoor[1] | indoor[2] | 0x00 | 0x00 | 0xA1 | door[0] | door[1] | door[2] |

Door station ignores indoor byte values and hardware type (0xA1).
As soon as door station receives this command, it stops analog audio/video onto the bus.

# Bus Checksum

```C
uint8_t crc(uint8_t *command, uint8_t len) {
    uint8_t crc = 0;
    for(uint8_t i=0; i<len; i++) {
        crc += command[i];
    }
    return crc;
}
```
