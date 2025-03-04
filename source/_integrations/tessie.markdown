---
title: Tessie
description: Instructions on how to integrate Tessie within Home Assistant.
ha_category:
  - Binary Sensor
  - Button
  - Climate
  - Cover
  - Device Tracker
  - Lock
  - Media Player
  - Number
  - Sensor
  - Update
ha_release: 2024.1
ha_iot_class: Cloud Polling
ha_config_flow: true
ha_codeowners:
  - '@Bre77'
ha_domain: tessie
ha_platforms:
  - binary_sensor
  - button
  - climate
  - cover
  - device_tracker
  - lock
  - media_player
  - number
  - select
  - sensor
  - switch
  - update
ha_integration_type: integration
---

The Tessie integration exposes various commands and sensors from the Tesla vehicles connected to your [Tessie](https://my.tessie.com/) account.

## Prerequisites

You must have a [Tessie](https://my.tessie.com/) account and [access token](https://my.tessie.com/settings/api).

{% include integrations/config_flow.md %}

## Entities

### Binary sensor

The integration will create binary sensor entities for a variety of metrics related to your vehicles:

#### Charge state

- Battery charging
- Battery heater
- Preconditioning enabled
- Scheduled charging enabled
- Trip charging enabled

#### Climate state

- Auto seat climate left
- Auto seat climate right
- Auto steering Wheel climate
- Overheat protection enabled
- Overheat protection running

#### Vehicle state

- Dashcam recording
- Front driver window
- Front passenger window
- Rear driver window
- Rear passenger window
- Tire pressure warning front left
- Tire pressure warning front right
- Tire pressure warning rear left
- Tire pressure warning rear right
- User present

### Button

The integration will create button entities to control various aspects of the vehicle:

- Flash lights
- Homelink
- Honk horn
- Keyless driving
- Play fart
- Wake

### Climate

The integration will create a climate entity to control the vehicle's climate control system. This entity can:

- Change the driver's set temperature
- Change to one of the three keep modes: Keep, Dog, and Camp
- Turn on and off

The passenger set temperature is shown as a sensor but cannot be changed by Tessie.

### Cover

The integration will create a cover entity to control various aspects of your vehicles:

- Open/Close trunk
- Open/Close charge port
- Open frunk
- Vent/Closing windows

### Device tracker

The integration will create device tracker entities for the vehicle's current location and navigation destination.

### Lock

The integration will create a lock entity for each vehicle.

### Media Player

The integration will create media player entities to show what each vehicle is currently playing.

### Number

The integration will create number entities to control:

- Charge current
- Charge limit
- Speed limit

### Select

The integration will create a select entity to control each of the seat heaters. It allows you to set each seat heater to Off, Low, Medium, or High.

- Front left
- Front right
- Rear center (if installed)
- Rear left (if installed)
- Rear right (if installed)
- Third row left (if installed)
- Third row right (if installed)

### Sensor

The integration will create sensor entities for a variety of metrics related to your vehicles:

#### Charge state

- Battery level
- Battery range
- Charge energy added
- Charge rate
- Charger current
- Charger power
- Charger voltage

#### Climate state

- Driver temperature setting
- Inside temperature
- Outside temperature
- Passenger temperature setting

#### Drive state

- Power
- Shift state
- Speed

#### Vehicle state

- Odometer
- Online
- Tire pressure front left
- Tire pressure front right
- Tire pressure rear left
- Tire pressure rear right

### Switch

The integration will create switch entities to control various aspects of your vehicles:

- Charge
- Defrost mode
- Sentry mode
- Steering wheel heater
- Valet mode

### Update

The integration will show vehicle software updates and their installation progress.
