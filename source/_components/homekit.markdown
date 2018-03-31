---
layout: page
title: "HomeKit"
description: "Instructions on how to setup the HomeKit component in Home Assistant."
date: 2018-02-20 17:30
sidebar: true
comments: false
sharing: true
footer: true
ha_category: Voice
ha_release: 0.64
logo: apple-homekit.png
---

The `HomeKit` component allows you to forward entities from Home Assistant to Apple `HomeKit`, so they could be controlled from Apple `Home` app and `Siri`. Please make sure that you have read the [considerations](#considerations) listed below to save you some trouble later.

<p class="note warning">
  It might be necessary to install an additional package:  
  `$ sudo apt-get install libavahi-compat-libdnssd-dev`
</p>

{% configuration %}
  homekit:
    description: HomeKit configuration.
    required: true
    type: map
    keys:
      auto_start:
        description: Flag if the HomeKit Server should start automatically after the Home Assistant Core Setup is done. ([Disable Auto Start](#disable-auto-start))
        required: false
        type: boolean
        default: true
      port:
        description: Port for the HomeKit extension.
        required: false
        type: int
        default: 51827
      filter:
        description: Filter entities to available in the `Home` app. ([Configure Filter](#configure-filter))
        required: false
        type: map
        keys:
          include_domains:
            description: Domains to be included.
            required: false
            type: list
          include_entities:
            description: Entities to be included.
            required: false
            type: list
          exclude_domains:
            description: Domains to be excluded.
            required: false
            type: list
          exclude_entities:
            description: Entities to be excluded.
            required: false
            type: list
      entity_config:
        description: Configuration for specific entities. All subordinate keys are the corresponding entity ids to the domains, e.g. `alarm_control_panel.alarm`.
        required: false
        type: map
        keys:
          alarm_control_panel:
            description: Additional options for `alarm_control_panel` entities.
            required: false
            type: map
            keys:
              code:
                description: Code to arm or disarm the alarm in the frontend.
                required: false
                type: string
                default: ''
{% endconfiguration %}


## {% linkable_title Setup %}

To enable the `HomeKit` component in Home Assistant, add the following to your configuration file:

```yaml
# Example for HomeKit setup
homekit:
```

After Home Assistant has started, the entities specified by the filter are exposed to `HomeKit` if the are [supported](#supported-components). To add them:
1. Open the Home Assistant frontend. A new card will display the `pin code`.
1. Open the `Home` app.
2. Choose `Add Accessory`, than select `Don't Have a Code or Can't Scan?` and enter the `pin code`.
4. Confirm the you are adding an `Uncertified Accessory` by clicking on `Add Anyway`.
5. Follow the setup be clicking on `Next` and lastly `Done` in the top right hand corner.
6. The `Home Assistant` Bridge and the Accessories should now be listed in the `Home` app.

After the setup is completed you should be able to control your Home Assistant components through `Home` and `Siri`.


## {% linkable_title Considerations %}


### {% linkable_title Accessory ID %}

Currently this component uses the `entity_id` to generate a unique `accessory id (aid)` for `HomeKit`. The `aid` is used to identify a device and save all configurations made for it. This however means that if you decide to change an `entity_id` all configurations for this accessory made in the `Home` app will be lost.

### {% linkable_title Persistence Storage %}

Unfortunately `HomeKit` doesn't support any kind of persistence storage, only the configuration for accessories that are added to the `Home Assistant Bridge` are kept. To avoid problems it is recommended to use an automation to always start `HomeKit` with at least the same entities setup. If for some reason some entities are not setup, their config will be deleted. (State unknown or similar will not cause any issues.)

A common situation might be if you decide to disable parts of the configuration for testing. Please make sure to disable `auto start` and `turn off` the `Start HomeKit` automation (if you have one).


## {% linkable_title Disable Auto Start %}

Depending on your individual setup, it might be necessary to disable `Auto Start` for all accessories to be available for `HomeKit`. Only those entities that are fully setup when the `HomeKit` component is started, can be added. To start `HomeKit` when `auto_start: False`, you can call the service `homekit.start`.

This can be automated using an `automation`.

{% raw %}
```yaml
# Example for Z-Wave
homekit:
  auto_start: False

automation:
  - alias: 'Start HomeKit'
    trigger:
      - platform: event
        event_type: zwave.network_ready
    action:
      - service: homekit.start
```
{% endraw %}

{% raw %}
```yaml
# Example using a delay after start of Home Assistant
homekit:
  auto_start: False

automation:
  - alias: 'Start HomeKit'
    trigger:
      - platform: homeassistant
        event: start
    action:
      - delay: 00:05  # Waits 5 minutes
      - service: homekit.start
```
{% endraw %}


## {% linkable_title Configure Filter %}

To limit which entities are being exposed to `HomeKit`, you can use the `filter` parameter. By default no entity will be excluded. Keep in mind though that only supported components can be added. The filter will favor an include list over an exclude list (if one item is included, all others are automatically excluded).
{% raw %}
```yaml
# Example configuration entry that only includes two entities:
homekit:
  auto_start: False
  filter:
    include_entities:
      - alarm_control_panel.home_alarm
      - switch.open_garage
```
{% endraw %}


## {% linkable_title Supported Components %}

The following components are currently supported:

| Component | Type Name | Description |
| --------- | --------- | ----------- |
| alarm_control_panel | SecuritySystem | All security systems. |
| climate | Thermostat | All climate devices. |
| cover | WindowCovering | All covers that support `set_cover_position`. |
| light | Light | Support for `on / off`, `brightness` and `rgb_color`. |
| sensor | TemperatureSensor | All sensors that have `Celsius` and `Fahrenheit` as their `unit_of_measurement`. |
| sensor | HumiditySensor | All sensors that have `%` as their `unit_of_measurement` |
| switch / remote / input_boolean / script | Switch | All represented as switches. |


## {% linkable_title Error reporting %}

If you encounter any issues or bug and want to report them on `GitHub`, please follow these steps to make it easier for others to help and get your issue solved.

1. Enable debugging mode:
```yaml
logger:
  default: warning
  logs:
       homeassistant.components.homekit: debug
```
2. Reproduce the bug / problem you have encountered.
3. Stop Home Assistant and copy the log from the log file. That is necessary since some errors only get logged, when Home Assistant is being shutdown.
4. Follow this link: [home-assistant/issues/new](https://github.com/home-assistant/home-assistant/issues/new?labels=component: homekit) and open a new issue.
5. Fill out all fields and especially include the following information:
   - The configuration entries for `homekit` and the `component` that is causing the issue.
   - The log / traceback you have generated before.
   - Screenshots of the failing entity in the `states` panel.
