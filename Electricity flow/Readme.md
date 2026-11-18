# Electricity Flow blueprint

The Electricity Flow blueprint calculates energy distribution between solar , the  grid, household  and battery . It determines how much energy flows between each source and destination, and stores the calculated values in Home Assistant helpers for monitoring, energy tracking and further calculations.

---
## install the blueprint

**Link nog aanpassen**
[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3dfflueprint.yaml)

**create the helpers first** with the packages below or manually.  

Create an automation using the **Electricity Flow** blueprint.

- Select the configured input sensors, if using the package some tuning is still necessary.
- The output helpers, if using the package all should set correctly already.
- Save and enable the automation.
---
## Creating helpers

The blueprint needs some helpers to make it easier I created packages files for the input en output helpers for kWh calculation. Using the blueprint with power sensors as input is also possible.
So this part is optional but i advise you to follow way first before as start. 

### 1. Enable Packages

Make sure the following configuration is present in your `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

- If a `homeassistant:` section already exists, add `packages` to that section.

### 2. Install Package Files

Create a `packages` folder in your Home Assistant configuration directory:

```text
/config/
├── configuration.yaml
└── packages/
    ├── electricity_flow_inputs_kwh.yaml
    └── electricity_flow_output_kwh.yaml
```

- Copy both YAML files into the `packages` folder.

### 3. Configure Input Sensors

**Required: User configuration**

Open `electricity_flow_inputs_kwh.yaml` and update the 'source:' entity IDs to match your own Home Assistant installation.

- Configure the utility_meter sensors.

Make sure the source sensors use the correct units and measurement conventions. The solar and/battery are options you can create more of necessary or remove when not used.

The file `electricity_flow_output_kwh.yaml` defines the `input_number` helpers used to store the calculated energy flows. No editing needed. 

### 5. Validate and Restart

1. Go to **Developer Tools → YAML**.
2. Check the configuration.
3. Restart Home Assistant.

---


## How it works

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

### The  code 
The main code which you can play with the the template-editor
 
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



