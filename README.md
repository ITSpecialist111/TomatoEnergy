TomatoEnergy Configuration for Home Assistant
Welcome to the TomatoEnergy Configuration for Home Assistant! This project enables users of Tomato Energy to set up an Electricity Dashboard in Home Assistant, making it easier to monitor energy usage, optimize costs, and switch tariffs automatically—perfect for those with CT clamp sensors like sensor.modbus_grid_consumption.

🎯 Project Goals
This project aims to:

Monitor electricity usage and associated costs.
Automate tariff switching based on time of day.
Provide a visual representation of energy usage and costs through an intuitive dashboard.
Help users optimize energy usage for devices like house batteries, EVs, and large appliances.
📂 Repository Contents
File/Folder Name	Description
Charge Based on Bayesian Drop Rate	YAML configuration to calculate and monitor charges based on Bayesian models.
Configuration YAML	Core YAML configurations for sensors, automations, and dashboards.
README.md	You're here! This document provides an overview of the project, installation, and contribution guidelines.
Stop Charge Based on Bayesian Drop Rate	Advanced automation to stop charging activities based on Bayesian drop rate analysis.
Tariff Automation	Automations for switching tariffs based on the current time.
Tomato Dashboard	Pre-configured Home Assistant dashboard for visualizing electricity usage and costs.
🚀 Installation
Follow these steps to get started:

1. Pre-requisites
A Home Assistant instance installed and running.
A CT clamp sensor (e.g., sensor.modbus_grid_consumption) for monitoring grid consumption.
Basic familiarity with YAML for Home Assistant configuration.
2. Clone This Repository
bash
Copy code
git clone https://github.com/your-username/TomatoEnergy.git
3. Add Configuration Files
Copy the YAML files from this repository to your Home Assistant configuration directory:
Charge Based on Bayesian Drop Rate
Configuration YAML
Tariff Automation
4. Restart Home Assistant
After copying the files, restart your Home Assistant instance for the changes to take effect.
5. Set Up the Dashboard
Use the YAML from the Tomato Dashboard file to create a new dashboard in Home Assistant.
Navigate to Settings > Dashboards in Home Assistant, and paste the dashboard YAML.
6. Validate and Test
Verify that sensors and automations are working as expected.
Check the dashboard for accurate energy usage and cost visualization.
💡 Key Features
Sensors

Convert power consumption (kW to kWh) using ModBus integration.
Estimate daily electricity costs and determine current electricity rates automatically.
Automations

Switch tariffs based on time of day using predefined rules.
Optimize energy usage for off-peak and drop-rate times.
Dashboard

Visualize electricity usage, costs, and breakdowns with graphs, gauges, and tables.
Gain actionable insights into energy consumption trends.
🤝 Contributing
We welcome contributions to improve this project! Here’s how you can help:

1. Fork the Repository
Click the Fork button on the top right of this repository.

2. Make Your Changes
Add features, improve documentation, or fix bugs.
Test your changes locally.
3. Submit a Pull Request
Create a pull request with a clear description of your changes.
Ensure your code follows the repository's style and structure.
📖 Documentation
For detailed documentation on the sensors, automations, and dashboard configurations, refer to the Configuration YAML and Tomato Dashboard files.

🛠 Support
If you encounter issues or have questions, feel free to open an Issue in this repository. We’ll be happy to assist you!

📜 License
This project is licensed under the MIT License. Feel free to use, modify, and distribute it as needed.
