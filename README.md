# TomatoEnergy
This is configuration for Home Assistant and Tomato Energy. There's no APIs today but fixed tariffs. Once billed there are CSV's available for you to download from Tomato's "MyWatts" website to compare.

This configuration assumes you have a CT clamp for monitoring power at the electric meter! In my config this is: sensor.modbus_grid_consumption 


Home Assistant Electricity Dashboard
This documentation provides a complete guide for setting up an Electricity Dashboard in Home Assistant. The setup includes sensors, automations, helpers, and a customizable dashboard to monitor electricity usage, costs, and tariffs.

Sensors
1. ModBus Integration Sensor (kWh Calculation)
Convert kW to kWh using the integration platform.

yaml
Copy code
sensor:
  - platform: integration
    source: sensor.modbus_grid_consumption
    name: ModBus GH Grid Consumption kWh
    round: 3
2. Projected Daily Cost Sensor
Calculate the projected cost for the day based on current usage.

yaml
Copy code
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
3. Current Electricity Rate Sensor
Dynamically determine the current electricity rate based on tariff times.

yaml
Copy code
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
Automations
1. Tariff Switching Automation
Automatically updates the tariff based on the current time.

yaml
Copy code
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
Helpers
1. Utility Meter Helpers
Track energy usage across different tariffs using sensor.modbus_gh_grid_consumption_kwh as the source.

yaml
Copy code
utility_meter:
  electricity_usage:
    source: sensor.modbus_gh_grid_consumption_kwh
    cycle: daily
    tariffs:
      - off_peak
      - peak
      - drop_rate
Dashboard
Dashboard Configuration
Below is the complete YAML configuration for the Electricity Dashboard:

yaml
Copy code
views:
  - title: Electricity Dashboard
    path: electricity-dashboard
    icon: mdi:flash
    badges: []
    sections:
      - type: grid
        cards:
          - type: markdown
            content: >
              ## Electricity Usage and Costs Overview  
              A quick summary of energy usage and associated costs.

          - type: custom:apexcharts-card
            graph_span: 12h
            header:
              show: true
              title: Grid Consumption Overtime - kWh
            series:
              - entity: sensor.modbus_gh_grid_consumption_kwh
                name: Grid Consumption
            apex_config:
              chart:
                type: line

          - type: entity
            entity: sensor.modbus_gh_grid_consumption_kwh
            name: Total Grid Consumption (kWh)

          - type: markdown
            content: >
              ## Electricity Tariff Times and Rates  

              **Tariff:** Tomato Lifestyle Fixed TP October 2024-V1-Tariff-Battery  

              - **Off-Peak**: 12:00 AM - 6:00 AM  
                Cost: **5.004p/kWh**  
              - **Drop Rate**:  
                - 9:30 AM - 11:30 AM  
                - 10:00 PM - 12:00 AM  
                Cost: **14.0427p/kWh**  
              - **Peak**: All other times  
                Cost: **24.3128p/kWh**  

              - **Standing Charge**: **40.8155p/day**  

              *Tariffs automatically adjust based on the time of day.*

      - type: grid
        columns: 2
        cards:
          - type: gauge
            entity: sensor.total_electricity_cost
            name: Daily Electricity Cost (£)
            min: 0
            max: 50
            severity:
              green: 0
              yellow: 20
              red: 40

          - type: gauge
            entity: sensor.modbus_grid_consumption
            name: Live Grid Usage (kW)
            min: 0
            max: 100
            severity:
              green: 0
              yellow: 50
              red: 80

          - type: entity
            entity: sensor.projected_daily_cost
            name: Projected Daily Cost (£)

          - type: custom:apexcharts-card
            graph_span: 1h
            header:
              show: true
              title: Real-Time Grid Usage (kW)
            series:
              - entity: sensor.modbus_grid_consumption
                name: Live Grid Usage
            apex_config:
              chart:
                type: line

      - type: grid
        columns: 2
        cards:
          - type: custom:apexcharts-card
            graph_span: 24h
            chart_type: donut
            header:
              show: true
              title: Electricity Usage Breakdown
              show_states: true
            series:
              - entity: sensor.electricity_usage_off_peak
                name: Off Peak Usage
              - entity: sensor.electricity_usage_drop_rate
                name: Drop Rate Usage
              - entity: sensor.electricity_usage_peak
                name: Peak Usage
            apex_config:
              chart:
                type: donut
              labels:
                - Off Peak
                - Drop Rate
                - Peak

          - type: custom:apexcharts-card
            graph_span: 24h
            chart_type: donut
            header:
              show: true
              title: Electricity Costs Breakdown
              show_states: true
            series:
              - entity: sensor.off_peak_energy_cost
                name: Off Peak Cost
              - entity: sensor.drop_rate_energy_cost
                name: Drop Rate Cost
              - entity: sensor.peak_energy_cost
                name: Peak Cost
            apex_config:
              chart:
                type: donut
              labels:
                - Off Peak
                - Drop Rate
                - Peak
Feel free to copy this document into your GitHub repository. Let me know if you need any additional refinements! 😊


