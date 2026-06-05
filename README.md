# Template Climate Fork

Fork of [`jcwillox/hass-template-climate`](https://github.com/jcwillox/hass-template-climate).

For all original configuration options, see the upstream README:
<https://github.com/jcwillox/hass-template-climate>

This fork only adds `turn_on` and `turn_off` actions.

## Added Options

| Name | Description |
| --- | --- |
| `turn_on` | Runs when `climate.turn_on` is called, or when `climate.toggle` turns the entity on. |
| `turn_off` | Runs when `climate.turn_off` is called, or when `climate.toggle` turns the entity off. |

When `turn_on` is configured, the integration runs that action instead of Home Assistant's default climate turn-on fallback. This lets an underlying climate device power on without forcing a new HVAC mode.

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

    turn_off:
      - service: climate.turn_off
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

## Behavior

- `climate.turn_on` / `toggle on`: runs `turn_on`, without forcing `heat` or `cool`.
- `climate.turn_off` / `toggle off`: runs `turn_off`.
- Selecting a mode such as `cool`, `dry`, or `heat`: still runs `set_hvac_mode`.

The underlying device must support power-on without forcing a fixed mode if you want it to keep its previous mode.
