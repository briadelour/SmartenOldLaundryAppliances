# SmartenOldLaundryAppliances

TP-Link KASA SmartPlug KP115
https://www.amazon.com/dp/B08LN3C7WK

Zigbee Vibration Sensor
https://www.amazon.com/dp/B0C9DP249C

Set Vibration Sensor dip switches for noise and sensitivity to:
SIREN: 1 OFF (since you will be alerting through Home Assistant and it won't burn through the batteries faster)
LEVEL 1: set to 0
LEVEL 2: set to 1
This sets the sensitivity to H for High.

Zigbee USB Dongle Receiver (if needed)
https://www.amazon.com/Universal-Assistant-Zigbee2MQTT-Coordinator-Automation/dp/B0FNLP58JS?th=1

## Washer cards
### Simple Button for Washer Electricity Usage
```yaml
type: tile
entity: sensor.washer_current
name: Washer Electric Draw
icon: mdi:washing-machine
show_entity_picture: false
hide_state: false
vertical: false
features_position: bottom
card_mod:
  style: |
    .icon {
     {% if states(sensor.washer_current)|float(0) > 2.00 %}
      --tile-color: red;
     {% elif states(sensor.washer_current)|float(0) > 1.00 %}
      --tile-color: orange;
     {% elif states(sensor.washer_current)|float(0) > 0.02 %}
      --tile-color: green;
     {% elif states(sensor.washer_current)|float(0) > 0.01 %}  
      --tile-color: blue;
     {% else %}  
      --tile-color: lightblue;
     {% endif %}  
    }
```
### Simple Waher Electricity Usage Graph
```yaml
chart_type: line
period: hour
type: statistics-graph
entities:
  - sensor.washer_current
stat_types:
  - max
  - mean
hide_legend: false
logarithmic_scale: false
title: Washer Electricity 12 hours
days_to_show: 0.5
```
## Dryer cards

```yaml
type: custom:mini-graph-card
entities:
  - entity: binary_sensor.dryer_vibration_sensor_motion
    name: Dryer Vibration
name: Dryer Motion last 6 hours
hours_to_show: 6
points_per_hour: 60
update_interval: 30
aggregate_func: max
line_width: 1
smoothing: true
state_map:
  - value: "off"
    label: Clear
  - value: "on"
    label: Detected

```
## Combined Simple card
```yaml
type: entities
entities:
  - type: custom:bar-card
    entity: sensor.washer_current
    name: Washing Machine Power Draw
    positions:
      icon: "off"
      value: inside
    style:
      top: 6.5%
      left: 10%
      background: none
      box-shadow: none
      text-shadow: none
      transform: none
      overflow: hidden
      border-radius: 0px
      width: 38%
    height: 25px
    min: 0
    max: 4
    severity:
      - color: Green
        from: 0
        to: 1
      - color: Orange
        from: 1.1
        to: 2.9
      - color: Red
        from: 3
        to: 7.5
  - entity: binary_sensor.dryer_vibration_sensor_motion
    name: Dryer Vibration
  - entity: sensor.dryer_vibration_sensor_battery
title: Laundry Room
state_color: true
```

## Automation Ideas
### Washer
Notification when current on washer sensor stays above 0.02 for 3 minutes, clear it when it drops below 0.03 for 2 minutes.
### Dryer
Check the binary sensor for the vibration sensor and alert when dryer has changed from on to off for at least a minute.
