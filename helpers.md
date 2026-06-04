# Required helper entities

## Charge every n days to 100%
_Try to find a good value for your battery. **10** is fine for the SMA Home Storage.
That means that every 10 days, the battery is charged to 100%, if there is enough solar energy.
The longer the battery spends without reaching 100%, the more uncalibrated its battery management system might become._

- Helper type: Number
- Entity ID: input_number.charge_every_n_days_to_100
- Minimum value: 1
- Maximum value: 31
- Display mode: Input field
- Step size: 1

## Recently charged to 100%
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


## Charge to 100% capacity multiplier
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

## Remaining charge goal today
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
- Step value: 1
- Unit of measurement: Wh

## input_number.battery_max_discharge_power

## [WIP: Add other entities from max. grid feed in]

## Maximum grid feed in for charging battery
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
      {% if ns.surplus_energy < (states('number.remaining_charge_goal_today') | int + states('input_number.charge_goal_overestimation') | int) -%}
        {{ w_grid_feed_in }}
        {% break -%}
      {% endif -%}
    {%- endfor %}
  {% endif -%}
  ```

- Minimum value: 0
- Maximum value: [Equal to the maximum output of the inverter, e.g. 3600]
- Step value: 1
- Unit of measurement: W
