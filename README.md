# Smart Laundry Monitor for Old Appliances

Transform your existing washer and dryer into smart appliances using affordable sensors and Home Assistant. Get notifications when your laundry is done without buying new machines!

<img width="386" height="515" alt="image" src="https://github.com/user-attachments/assets/3adc7246-647b-4d66-b4e3-bd7ea10c6f7d" />

## Overview

This project uses a smart plug to monitor electricity usage for the washing machine and a vibration sensor for the dryer. When integrated with Home Assistant, you'll receive notifications when laundry cycles complete, allowing you to move loads promptly and avoid that musty smell or wrinkled clothes.

**Total Cost:** ~$40-60 depending on whether you need a Zigbee coordinator

## Hardware Required

### Required Components

| Component | Purpose | Link |
|-----------|---------|------|
| **TP-Link KASA Smart Plug (KP115)** | Monitors washer power consumption with built-in energy monitoring | [Amazon](https://www.amazon.com/dp/B08LN3C7WK) |
| **Zigbee Vibration Sensor** | Detects dryer vibrations during operation | [Amazon](https://www.amazon.com/dp/B0C9DP249C) |

### Optional Component (if you don't already have one)

| Component | Purpose | Link |
|-----------|---------|------|
| **Zigbee USB Dongle** | Required only if you don't already have a Zigbee coordinator in your Home Assistant setup | [Amazon](https://www.amazon.com/Universal-Assistant-Zigbee2MQTT-Coordinator-Automation/dp/B0FNLP58JS?th=1) |

## Prerequisites

- Home Assistant installation (Home Assistant OS, Container, or Supervised)
- WiFi network (for the TP-Link smart plug)
- Zigbee integration configured in Home Assistant (Zigbee2MQTT, ZHA, or similar)

## Installation

### 1. Configure the Vibration Sensor

Before installing the vibration sensor on your dryer, configure the DIP switches for optimal battery life and sensitivity:

**DIP Switch Settings:**
- **SIREN:** OFF (Position 1) - Disables the built-in siren since Home Assistant will handle notifications, preserving battery life
- **LEVEL 1:** 0 (OFF)
- **LEVEL 2:** 1 (ON)

These settings configure the sensor to **High (H)** sensitivity, which works well for detecting dryer vibrations.

### 2. Install the Smart Plug

1. Plug the TP-Link KP115 into the wall outlet behind your washing machine
2. Plug your washing machine into the smart plug
3. Follow the Kasa app instructions to set up the plug on your WiFi network
4. Integrate the plug into Home Assistant using the TP-Link Kasa integration

### 3. Install the Vibration Sensor

1. Pair the vibration sensor with your Zigbee network through Home Assistant
2. Attach the sensor to the top or side of your dryer where vibrations are most noticeable
3. Test by running the dryer briefly and checking that motion is detected in Home Assistant

## Home Assistant Configuration

### Washer Monitoring Cards

#### Power Consumption Tile
Displays current power draw with color-coded status indicators:

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
     {% if states('sensor.washer_current')|float(0) > 2.00 %}
      --tile-color: red;
     {% elif states('sensor.washer_current')|float(0) > 1.00 %}
      --tile-color: orange;
     {% elif states('sensor.washer_current')|float(0) > 0.02 %}
      --tile-color: green;
     {% elif states('sensor.washer_current')|float(0) > 0.01 %}  
      --tile-color: blue;
     {% else %}  
      --tile-color: lightblue;
     {% endif %}  
    }
```

**Color Guide:**
- 🔴 Red: Heavy load (>2.00A)
- 🟠 Orange: Normal wash (1.00-2.00A)
- 🟢 Green: Light operation (0.02-1.00A)
- 🔵 Blue: Standby (0.01-0.02A)
- 💙 Light Blue: Off (<0.01A)

#### Power Consumption Graph
12-hour historical view of washer electricity usage:

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

### Dryer Monitoring Card

#### Vibration History Graph
6-hour view of dryer activity:

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

### Combined Laundry Status Card

Single card showing both appliance statuses:

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

## Automation Examples

### Washer Complete Notification

Sends a notification when the washer finishes its cycle:

**Trigger:** Current draw stays above 0.02A for 3 minutes (cycle running)  
**Action:** Wait until current drops below 0.03A for 2 minutes, then send notification  

**Example automation:**
```yaml
alias: Washer Cycle Complete
trigger:
  - platform: numeric_state
    entity_id: sensor.washer_current
    below: 0.03
    for:
      minutes: 2
condition:
  - condition: numeric_state
    entity_id: sensor.washer_current
    above: 0.02
    for:
      minutes: 3
action:
  - service: notify.mobile_app
    data:
      title: "Laundry Alert"
      message: "Washing machine cycle is complete!"
```

### Dryer Complete Notification

Sends a notification when the dryer cycle finishes:

**Trigger:** Vibration sensor changes from 'on' to 'off' for at least 1 minute  
**Action:** Send notification that dryer has stopped

**Example automation:**
```yaml
alias: Dryer Cycle Complete
trigger:
  - platform: state
    entity_id: binary_sensor.dryer_vibration_sensor_motion
    from: "on"
    to: "off"
    for:
      minutes: 1
action:
  - service: notify.mobile_app
    data:
      title: "Laundry Alert"
      message: "Dryer cycle is complete!"
```

## Troubleshooting

### Washer Not Detected
- Verify the smart plug is properly connected to WiFi
- Check that the plug is integrated in Home Assistant
- Ensure your washer draws enough power to register (>0.02A)

### Dryer Sensor Not Responding
- Check battery level in Home Assistant
- Verify the sensor is firmly attached to the dryer
- Confirm Zigbee coordinator is within range
- Try adjusting sensor sensitivity using DIP switches

### False Notifications
- Adjust timing thresholds in automations
- For washer: increase the "below threshold" time
- For dryer: increase the "off state" duration before notification

## Future Enhancements

- Add washing machine door sensor to detect when laundry is removed
- Integrate with voice assistants (Alexa, Google Home)
- Track laundry statistics (cycles per week, energy usage)
- Create dashboard with historical data and trends
- Add second vibration sensor to washer for dual monitoring

## Contributing

Found a better configuration or have suggestions? Feel free to open an issue or submit a pull request!

## License

This project is open source and available under the MIT License.

## Acknowledgments

Built with ❤️ for people tired of forgetting about wet laundry in the washer.
