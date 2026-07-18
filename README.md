THIS IS AN INCOMPLETE REPO. ONCE THE FIRST VERSION IS COMPLETE, THIS LINE WILL BE REMOVED.

# HA-battery-controller-SBSE
This is a Home Assistant automation + peripherals to control a battery via the hybrid SMA Sunny Boy Smart Energy inverter.

**What to expect:**
After following the instructions in this repo, your battery should be charged at the middle of the day. This reduces pressure on the energy grid, and increases self-consumption when energy is cheapest. Optionally, your battery will be kept healthy by limiting how often it is charged to 100%.

**Requirements:**
- A Sunny Boy Smart Energy inverter, with a battery (currently only tested with SMA Home Storage)
- Readings of the energy meters P1 port with [the DSMR integration](https://www.home-assistant.io/integrations/dsmr)
- A well-configured [Open-Meteo Solar Forecast integration](https://github.com/rany2/ha-open-meteo-solar-forecast)

If your setup does not meet these requirements, well, set it up first! Or continue reading to take inspiration for setting up a similar automation in your Home Assistant instance.

**Setup:**
1. Enable Modbus communication to the inverter (see below for how)
2. Add the configuration in `modbus-configuration.yaml` to the `configuration.yaml` in Home Assistant
3. Restart Home Assistant to enable the Modbus integration.
4. Create the entities described in entities.md
5. Create an automation with the 'Battery SoC 100% marker' blueprint, and fill according to the instructions in the blueprint
6. Create an automation with the 'SMA SBSE Battery control' blueprint, and fill according to the instructions in the blueprint
7. Enable external energy management on the inverter (see below for how)

## Inverter settings

**How to enable Modbus communication with the Sunny Boy Smart Energy**
1. Connect to the inverter through EnnexOS
2. Go to the inverter's device -> Configuration -> External communication -> Edit (pencil icon)
3. Toggle on 'Access for *Modbus TCP* enable' toggle on
4. Leave port on default 502
5. Leave 'Access for *Modbus UDP* enable' toggled off
6. Save

This is required for the Modbus integration of Home Assitant to read out the values from the inverter.

**How to enable inverter/battery control over Modbus with the Sunny Boy Smart Energy**
1. Connect to the inverter through EnnexOS
2. Go to the plant's Configuration -> Energy managemennt -> Advanced Settings -> Edit
3. Select 'No control by SMA Energy Management'
4. Save

This is required for controlling the battery (dis)charge power from an external source, in this case Home Assistant.

## Trouble shooting

**Despite everything, the inverter seems to limit the battery (dis)charge power!**

Perhaps the maximum and minimum power levels for the battery are limited internally in the inverter.
To reset these values, switch the energy management back to SMA's own battery control. Set the energy profile to anything (for example maximum own consumption).
Wait for a minute or so. Then switch the energy management back to external control.
