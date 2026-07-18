# Required helper entities

## Independent helper entities
These helper entities are not relying on integrations, devices, or whatsoever. Helpers in other sections might be dependent on these entities. So, set the helpers from this section up first.

### Charge every n days to 100%
_Try to find a good value for your battery. **10** is fine for the SMA Home Storage.
That means that every 10 days, the battery is charged to 100%, if there is enough solar energy.
The longer the battery spends without reaching 100%, the more uncalibrated its battery management system might become._

- Helper type: Number
- Entity ID: input_number.charge_every_n_days_to_100
- Minimum value: 1
- Maximum value: 31
- Display mode: Input field
- Step size: 1

### Recently charged to 100%
- Helper type: Template > Binary sensor
- Entity ID: binary_sensor.recently_charged_to_100
- Template:

  ```
  {{
    (
      as_timestamp(today_at())
      - as_timestamp(states('input_datetime.last_date_charged_to_100'))
    ) / 3600 / 24
     < states('input_number.charge_every_n_days_to_100') | int 
  }}
  ```


### Charge to 100% capacity multiplier
_Charging from 99% to 100% usually takes longer than any other 1%-increase, becasue the battery management systems decalibrate overtime.
Hence, the remaining battery capacity is actually more than the 1% might imply.
To overcome this, the last 1% is inflated by this multiplier to charge with more power.
A value of **2** works fine for the SMA Home Storage.
If the battery gets stuck at 99%, then this multiplier can be increased._

- Helper type: Number
- Entity ID: input_number.charge_to_100_capacity_multiplier
- Minimum value: 1
- Maximum value: 3
- Display mode: Input field
- Step size: 0.01

### Maximum discharge power of battery
_The hard limit of the discharge power. At all times, the discharge power will be capped at this value.
Set it to either the maximum output of the system, or to a lower value, if desired._

- Helper type: Number
- Entity ID: input_number.battery_max_discharge_power
- Minimum value: 1
- Maximum value: [Maximum possible discharge power of the batteries, or the maximum power of the PV-system, whichever is lowest]
- Step size: 1
- Unit of measurement: W

### Battery charge setpoint
_This is the control variable for the battery (dis)charge.
Its value represents the power at which the battery is (dis)charged now.
It is updated every time the automation 'SMA SBSE Battery Control' is run.
Initialise with 0._
- Helper type: Number
- Entity ID: input_number.battery_charge_setpoint
- Minimum value: [Equal to the negative minimum discharge power of the battery, e.g. -3600]
- Maximum value: [Equal to the positive maximum charge power of the battery, e.g. 3600]
- Step size: 1
- Unit of measurement: W

## Battery-dependent helper entities
These helper entities are dependent on information from the battery. This information is gathered by the 'Modbus' integration. The entities from 'Modbus' that are required are:
- sensor.battery_soc

### Remaining charge goal today
- Type: Template > Number
- Entity ID: number.remaining_charge_goal_today
- Template:

  ```
  {% if states('binary_sensor.recently_charged_to_100') | bool %}
    {{ 6500 * max(0, ((states('input_number.maximum_soc_for_charging') | int - states('sensor.battery_soc') | int) / 100 ))}}
  {% else %}
    {{ 6500.0 * states('input_number.charge_to_100_capacity_multiplier') | float * ((100 - states('sensor.battery_soc') | int) / 100 )}}
  {% endif %}
  ```

- Minimum value: 0
- Maximum value: [Total energy capacity of battery in Wh, e.g. 6500]
- Step size: 1
- Unit of measurement: Wh

## DSMR-smart meter integration-dependent entities
These helpers use entities from the device 'Energy Meter' from the integration 'DSMR Smart Meter'. From this device, the following entities are used:
- sensor.energy_production_today_2
- sensor.electricity_meter_power_consumption
- sensor.electricity_meter_power_production

### Maximum grid feed-in for charging battery
_This template calculates the power at which the PV production should be returned to the grid, in order to have enough energy left to charge the battery throughout the day, to the desired SoC.
If the desired SoC is reached, this value will have the value of `input_number.battery_max_discharge_power`, because all surplus produced energy can be fed to the grid._

- Helper type: Template > Number
- Entity ID: number.maximum_grid_feed_in_for_charging_battery
- Template:

  ```
  {% if states('number.remaining_charge_goal_today') | int <= 0 %}
    {{ states('input_number.battery_max_discharge_power') }}
  {% else %}
    {% set ns = namespace(surplus_energy=0)  %}
    {% for w_grid_feed_in in range(0, 3600, 50) -%}
      {%- set ns.surplus_energy = 0 -%}
      
      {%- for key in state_attr('sensor.energy_production_today_2', 'wh_period').keys() -%}
        {% if as_datetime(key) >= now() %}
          {%- set ns.surplus_energy = ns.surplus_energy | int + max(state_attr('sensor.energy_production_today_2', 'wh_period')[key] | int - w_grid_feed_in | int, 0) -%}
        {% endif %}
      {%- endfor %}
      {% if ns.surplus_energy < states('number.remaining_charge_goal_today') | int -%}
        {{ w_grid_feed_in }}
        {% break -%}
      {% endif -%}
    {%- endfor %}
  {% endif -%}
  ```

- Minimum value: 0
- Maximum value: [Equal to the maximum output of the inverter, e.g. 3600]
- Step size: 1
- Unit of measurement: W

### Netto energy consumption
- Helper type: Template > Sensor
- Entity ID: sensor.netto_energy_consumption
- Template:

  ```
  {{ states('sensor.electricity_meter_power_consumption') | int - states('sensor.electricity_meter_power_production') | int }}
  ```

- Device class: Power
- State class: Measurement
- Unit of measurement: W

