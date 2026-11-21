# Electricity Flow blueprint

> **This project has moved.** Electricity Flow is now a custom integration (installable via HACS) in its own repository:
> **[github.com/Tazzios/ha-electricity-flow](https://github.com/Tazzios/ha-electricity-flow)**.
> The blueprint and package below are no longer maintained.

The Electricity Flow blueprint calculates energy distribution between solar , the  grid, household  and battery. It determines how much energy flows between each source and destination, and stores the calculated values in Home Assistant helpers for monitoring, energy tracking and further calculations.

My use cases is to calculate the savings from my battery and solar.
Pricing that i want to set:  
- Solar to home, grid import prices.
- Solar to battery, grid export prices.
- Battery to home, grid import prices.  
- Battery to home, grid export prices.  

Note: Because of the 15 minute aggregation it won`t be 100% accurate but smaller will have other issues.

More details about wat it does and how it works in the [Technical design document](https://github.com/Tazzios/Home-Assistant/blob/main/Electricity%20flow/Technical%20Design%20Document.md) document.

---
## install the blueprint

1. Install the blueprint:
[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FTazzios%2FHome-Assistant%2Fblob%2Fmain%2FElectricity%20flow%2Felectricity%20flow.yaml)

2. **create the helpers first** with the packages below or manually.  

3. Create an automation using the **Electricity Flow** blueprint.

- Select the configured input sensors, if using the package some tuning is still necessary.
- The output helpers, if using the package all should set correctly already.
- Save and enable the automation.
---
## Creating helpers

The blueprint needs some helpers to make it easier I created packages files for the input en output helpers for kWh calculation. Using the blueprint with power sensors as input is also possible.
So this part is optional but i advise you to follow way first before as start. 

**Manually**

1.Create utility meter sensors set on 15 minutes for each input that you want to add.  
2. Create input_numbers* for each output that you want.

 
<sup>* A blueprint cannot write in a sensor</sup>

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





