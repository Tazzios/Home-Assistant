# Technical Design documentation

## How does it work

This is an explanation for the use with energy(kWh) sensors not with power(W).
 
1. The utility meters counts the kWh to 15 minute data
2. The blueprint calculates the energy flow and write them in the output

## Possible Improvements
Because the use of a blueprint there where some limitation. When this would be converted to an integration the manual creation of the in en outputs would not be necessary. Maybe even a price calculation could directly added, currently i use [Dynamic Energy Cost](https://github.com/martinarva/dynamic_energy_cost/) to do this.

I currently do not have the skills to convert it to a Integration.

---
## energy distribution calculation

The blueprint calculates how energy is distributed. These calculations are based on measured sensor values and an assumed distribution priority. The blueprint follows these steps:

### 1. Solar Distribution

Solar production is allocated in the following order:

1. **Solar → Home:** Solar energy is used to cover household consumption first.
2. **Solar → Battery:** Remaining solar surplus is used to charge the battery.
3. **Solar → Grid:** Any surplus left after battery charging is exported to the electricity grid.

### 2. Battery Distribution

Battery discharge is distributed according to measured grid export and remaining household demand:

1. **Battery → Grid:** Battery energy exported to the electricity grid.
2. **Battery → Home:** Remaining battery discharge is allocated to household consumption.

### 3. Grid Distribution

Grid import and battery charging are used to determine the remaining grid energy flows:

1. **Grid → Battery:** Battery charging not covered by solar surplus.
2. **Grid → Home:** Remaining household consumption not covered by solar or battery energy.

### The code 
The main code which you can play with the the template-editor:  
(If you see weird out double check if your input is realistic)
 
```YAML
{% set solar_production = 0.500 | float(0) %}
{% set grid_power = 0.000 | float(0) %}
{% set battery_charge = 0.00 | float(0) %}
{% set battery_discharge = 0.200 | float(0) %}


{# --- Grid import/export scheiden --- #}
{% set grid_import = max(grid_power, 0) %}
{% set grid_export = max(-grid_power, 0) %}

{# --- Huisverbruik afleiden --- #}
{% set house_consumption = max(
  solar_production + battery_discharge
  + grid_import - battery_charge - grid_export, 0
) %}

{# --- Zon eerst naar huis --- #}
{% set solar_to_home = min(solar_production, house_consumption) %}

{# --- Resterende zonne-energie --- #}
{% set solar_surplus = max(solar_production - solar_to_home, 0) %}

{# --- Zon naar batterij --- #}
{% set solar_to_battery = min(battery_charge, solar_surplus) %}

{# --- Zon naar grid --- #}
{% set solar_to_grid = max(solar_surplus - solar_to_battery, 0) %}

{# --- Batterij naar grid --- #}
{% set battery_to_grid = min(battery_discharge, grid_export) %}

{# --- Batterij naar huis --- #}
{% set battery_to_home = min(
  max(battery_discharge - battery_to_grid, 0),
  max(house_consumption - solar_to_home, 0)
) %}

{# --- Grid naar batterij --- #}
{% set grid_to_battery = max(battery_charge - solar_to_battery, 0) %}

{# --- Grid naar huis --- #}
{% set grid_to_home = max(
  house_consumption - solar_to_home - battery_to_home, 0
) %}

{# --- RESULTATEN --- #}
Zon → huis: {{ solar_to_home | round(3) }}
Zon → grid: {{ solar_to_grid | round(3) }}
Zon → batterij: {{ solar_to_battery | round(3) }}
Grid → huis: {{ grid_to_home | round(3) }}
Grid → batterij: {{ grid_to_battery | round(3) }}
Batterij → huis: {{ battery_to_home | round(3) }}
Batterij → grid: {{ battery_to_grid | round(3) }}
```
