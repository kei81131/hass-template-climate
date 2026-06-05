# Template Climate Fork

Fork of [`jcwillox/hass-template-climate`](https://github.com/jcwillox/hass-template-climate).

For all original configuration options, see the upstream README:
<https://github.com/jcwillox/hass-template-climate>

This fork only adds `turn_on` action.

## Added Options

| Name | Description |
| --- | --- |
| `turn_on` | Runs when `climate.turn_on` is called, or when `climate.toggle` turns the entity on. |

When `turn_on` is configured, it only power on the climate entity with last HVAC mode and  
does not force a new HVAC mode.

## Example

```yaml
climate:
  - platform: climate_template
    name: bedroom_ac
    unique_id: bedroom_ac

    turn_on:
      - service: climate.turn_on
        target:
          entity_id: climate.your_climate_entity

    hvac_mode_template: "{{ states('climate.your_climate_entity') }}"
    current_temperature_template: "{{ states('sensor.your_temp_entity') }}"
    current_humidity_template: "{{ states('sensor.your_humidity_entity') }}"

    modes:
      - "auto"
      - "heat"
      - "dry"
      - "off"
      - "cool"
      - "fan_only"

    set_hvac_mode:
      - service: climate.set_hvac_mode
        target:
          entity_id: climate.your_climate_entity
        data:
          hvac_mode: "{{ hvac_mode }}"

    set_temperature:
      - service: climate.set_temperature
        target:
          entity_id: climate.your_climate_entity
        data:
          temperature: "{{ temperature }}"
```


