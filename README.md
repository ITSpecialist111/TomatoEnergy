# TomatoEnergy Dashboard for Home Assistant

This repository provides a step-by-step guide for setting up a custom Electricity Monitoring Dashboard in Home Assistant. It is tailored for users of Tomato Energy who want to monitor their energy usage, costs, and tariffs without relying on APIs.

## Goals of the Project

* **📊 Real-Time Monitoring**: Track grid consumption with a CT clamp sensor (e.g., sensor.modbus_grid_consumption)
* **💸 Cost Breakdown**: Visualize energy usage and associated costs per tariff
* **🔋 Dynamic Charging**: Automate EV and battery charging based on time and Bayesian analysis
* **🏠 Beginner-Friendly**: Designed for users with no programming experience—just copy and paste YAML into Home Assistant

## Table of Contents

* Overview of Files
* Step-by-Step Setup
* Create Helpers
* Add Sensors
* Set Up Automations
* Configure the Dashboard
* Set Up Bayesian Charging Automation
* Contributing
* License

## Overview of Files

This repository contains the following files:

* **Configuration YAML**: Includes sensors for energy usage, costs, and tariffs  https://github.com/ITSpecialist111/TomatoEnergy/blob/main/Configuration%20YAML 
* **Tariff Automation**: Automates tariff switching based on the time of day  https://github.com/ITSpecialist111/TomatoEnergy/blob/main/Tariff%20Automation
* **Tomato Dashboard**: Full dashboard YAML for monitoring usage and costs https://github.com/ITSpecialist111/TomatoEnergy/blob/main/Tomato%20Dashboard 
* **Charge Based on Bayesian Drop Rate**: Automates charging logic for EVs or house batteries  https://github.com/ITSpecialist111/TomatoEnergy/blob/main/Charge%20Based%20on%20Bayesian%20Drop%20Rate

## Step-by-Step Setup

Follow these steps to set up the TomatoEnergy Dashboard.

### 1. Create Helpers

Helpers are used to organize energy usage into specific time-based tariffs.

1. Go to Settings > Devices & Services > Helpers in Home Assistant
2. Create a Utility Meter helper with the following configuration:
   * Source Entity: sensor.modbus_grid_consumption_kwh   (What ever sensor or CT Clamp you want to reference here that can see meter usage)
   * Cycle: Daily
   * Tariffs:
     * Off-Peak
     * Drop Rate
     * Peak
3. Repeat this process for all tariffs

### 2. Add Sensors

Add the following YAML code to your configuration.yaml. These sensors calculate energy usage, costs, and tariffs.

#### ModBus Integration Sensor
Converts power usage from kW to kWh. ONLY IF YOU DON'T ALREADY HAVE A SENSOR THAT'S IN kWH. I didn't, the sensor was in kW, so I needed to do a conversion. You may want to skip this step.

```yaml
sensor:
  - platform: integration
    source: sensor.modbus_grid_consumption
    name: ModBus GH Grid Consumption kWh
    round: 3
```

#### Projected Daily Cost Sensor
Estimates daily electricity costs based on current usage.

```yaml
template:
  - sensor:
      - name: "Projected Daily Cost"
        unit_of_measurement: "£"
        state: >
          {% set cost = states('sensor.total_electricity_cost') | float %}
          {% set current_hour = now().hour | float %}
          {% if current_hour > 0 %}
            {{ ((cost / current_hour) * 24) | round(2) }}
          {% else %}
            0.00
          {% endif %}
```

#### Current Electricity Rate Sensor
Determines the current tariff rate dynamically based on the time.

```yaml
template:
  - sensor:
      - name: "Current Electricity Rate Tomato"
        unique_id: current_electricity_rate_tomato
        unit_of_measurement: "p/kWh"
        state: >
          {% set off_peak_rate = 0.05004 %}
          {% set drop_rate = 0.140427 %}
          {% set peak_rate = 0.243128 %}
          {% set current_time = now().strftime("%H:%M") %}

          {% if "00:00" <= current_time < "06:00" %}
            {{ off_peak_rate }}
          {% elif "09:30" <= current_time < "11:30" %}
            {{ drop_rate }}
          {% elif "22:00" <= current_time < "23:59" %}
            {{ drop_rate }}
          {% else %}
            {{ peak_rate }}
          {% endif %}
```

After adding the sensors, restart Home Assistant.

### 3. Set Up Automations

Use automations to dynamically update tariffs. Copy the following YAML into your automations:

#### Tariff Automation
Switches between tariffs based on the time of day.

```yaml
alias: Update Electricity Tariff
trigger:
  - platform: time_pattern
    minutes: "/5"  # Runs every 5 minutes
action:
  - service: utility_meter.select_tariff
    target:
      entity_id: utility_meter.electricity_usage
    data:
      tariff: >
        {% set current_time = now().strftime("%H:%M") %}
        {% if "00:00" <= current_time < "06:00" %}
          off_peak
        {% elif "09:30" <= current_time < "11:30" %}
          drop_rate
        {% elif "22:00" <= current_time < "23:59" %}
          drop_rate
        {% else %}
          peak
        {% endif %}
```

### 4. Configure the Dashboard

The dashboard provides a visual overview of energy usage and costs. To configure the dashboard:

1. Navigate to Settings > Dashboards > Raw Configuration Editor
2. Copy and paste the YAML from Tomato Dashboard
3. Save and reload the dashboard

### 5. Set Up Bayesian Charging Automation

This setup uses a Bayesian sensor to determine the optimal times to charge EVs or batteries based on the current tariff and grid load.

#### Bayesian Sensor Configuration
Add the following YAML to your configuration.yaml:

```yaml
bayesian:
  - platform: bayesian
    name: "Charging Decision"
    prior: 0.5
    probability_threshold: 0.9
    observations:
      - platform: state
        entity_id: sensor.current_electricity_rate_tomato
        to_state: "0.05004"
        probability: 0.9
      - platform: state
        entity_id: sensor.grid_consumption
        below: 2.0
        probability: 0.8
```

#### Bayesian Automation
Create an automation to start charging when the Bayesian sensor outputs on:

```yaml
alias: Start EV Charging
trigger:
  - platform: state
    entity_id: binary_sensor.charging_decision
    to: "on"
action:
  - service: switch.turn_on
    target:
      entity_id: switch.ev_charger
```

Create another automation to stop charging when the Bayesian sensor outputs off:

```yaml
alias: Stop EV Charging
trigger:
  - platform: state
    entity_id: binary_sensor.charging_decision
    to: "off"
action:
  - service: switch.turn_off
    target:
      entity_id: switch.ev_charger
```

## Contributing

We welcome contributions! To contribute:

1. Fork the repository
2. Make your changes
3. Open a pull request with a detailed explanation of your updates

## License

This project is open-source and available under the MIT License.
