# HA-battery-controller-SBSE
This is a Home Assistant automation + peripherals to control a battery via the hybrid SMA Sunny Boy Smart Energy inverter.

**Requirements:**
- An entity that represents the netto power consumption at the grid connection. It has to be a live value, so not something polled from the cloud. For example, read the P1 port with [the DSMR integration](https://www.home-assistant.io/integrations/dsmr).
- Have a well-configured solar forecast integration running, for example [Open-Meteo Solar Forecast](https://github.com/rany2/ha-open-meteo-solar-forecast).

**Setup:**
1. Add the configuration in modbus-configuration.yaml to the configuration.yaml in Home Assistant.
2. Create an automation with the 'Battery SoC 100% marker' blueprint.
3. Create the entities described in entities.md

WIP. Still left to add:

5. List other necessary entities for battery control autmation
6. Upload autmation blueprints
7. Share this

Tips:
- How do I get a 'netto energy consumption' entity from [the DSMR integration](https://www.home-assistant.io/integrations/dsmr)?
    - Create a template helper with template:
      
      ```{{ states('sensor.electricity_meter_power_consumption') | int - states('sensor.electricity_meter_power_production') | int }}```
    - Unit of measurement: W
    - Device class: Power
    - State class: Measurement
    - 
