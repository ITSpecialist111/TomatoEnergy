# Step-by-Step Guide to Setting Up a Custom Tomato Energy Monitoring Dashboard in Home Assistant

## Project Overview
Welcome to the TomatoEnergy Dashboard project! This guide provides step-by-step instructions for setting up a custom Electricity Monitoring Dashboard in Home Assistant. The focus is on monitoring Tomato Energy's fixed tariffs, optimizing energy usage, and automating dynamic tariff changes.

## Project Goals
* **Real-Time Monitoring**: Track grid consumption with a CT clamp sensor (e.g., sensor.modbus_grid_consumption)
* **Cost Breakdown**: Visualize energy usage and associated costs per tariff
* **Dynamic Charging**: Automate EV and battery charging based on time and tariffs
* **Beginner-Friendly**: No APIs are required—just copy and paste YAML into Home Assistant

## How to Use This Repository
This repository provides individual YAML files for configuration. You can copy the required sections into your Home Assistant setup.

### Files Overview
* **Configuration YAML**: All sensor configurations
* **Tariff Automation**: Automates tariff switching based on time
* **Tomato Dashboard**: Complete dashboard YAML for monitoring
* **Charge Based on Bayesian Drop Rate**: Automates EV or battery charging

## Step-by-Step Setup

### Add Helpers in Home Assistant
1. Navigate to Settings > Devices & Services > Helpers
2. Create a Utility Meter helper:
   * Source Entity: sensor.modbus_grid_consumption_kwh
   * Cycle: Daily
   * Tariffs:
     * Off-Peak
     * Drop Rate
     * Peak
3. Repeat this process for each tariff

### Add Sensors
Copy the relevant YAML from the Configuration YAML file into your configuration.yaml.

#### Example Sensors

##### ModBus Integration Sensor
```yaml
sensor:
  - platform: integration
    source: sensor.modbus_grid_consumption
    name: ModBus GH Grid Consumption kWh
    round: 3
```

##### Projected Daily Cost
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

##### Current Electricity Rate
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
            off_peak
          {% elif "09:30" <= current_time < "11:30" %}
            drop_rate
          {% elif "22:00" <= current_time < "23:59" %}
            drop_rate
          {% else %}
            peak
          {% endif %}
```

#### Example Automation
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

### Dashboard Setup
The dashboard provides a visual overview of energy usage and costs. Copy the YAML from Tomato Dashboard into your dashboard's Raw Configuration Editor.

#### How to Add a Custom Dashboard
1. Go to Settings > Dashboards > Edit Dashboard
2. Select Raw Configuration Editor
3. Paste the contents of the Tomato Dashboard
4. Save and reload the dashboard

## Contributing
We welcome contributions! If you have ideas to improve this setup:

1. Fork the repository
2. Make your changes
3. Open a pull request explaining your updates

## License
This project is licensed under the MIT License.
