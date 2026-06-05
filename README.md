# Template Climate Fork

This fork is based on [`jcwillox/hass-template-climate`](https://github.com/jcwillox/hass-template-climate).

For the full integration documentation and all original configuration options, please use the upstream README:

<https://github.com/jcwillox/hass-template-climate>

This README only documents the extra `turn_on` and `turn_off` actions added by this fork.

## Extra Options

| Name | Type | Description |
| --- | --- | --- |
| `turn_on` | Home Assistant action | Runs when `climate.turn_on` is called, or when `climate.toggle` turns the entity on. |
| `turn_off` | Home Assistant action | Runs when `climate.turn_off` is called, or when `climate.toggle` turns the entity off. |

If `turn_on` is configured, this fork will run that action instead of Home Assistant's default climate `turn_on` fallback. This is useful when the underlying climate device already remembers its last mode and you only want to power it on without forcing `heat`, `cool`, or any other HVAC mode.

If either option is not configured, the integration falls back to the original upstream behavior for that service.

## Example

This example wraps an existing climate entity and makes `turn_on` / `turn_off` only call the wrapped entity's own power behavior. Manual mode changes still go through `set_hvac_mode`.

```yaml
climate:
  - platform: climate_template
    name: bedroom_ac
    unique_id: bedroom_ac

    turn_on:
      - service: climate.turn_on
        target:
          entity_id: climate.<your_climate_entity>

    turn_off:
      - service: climate.turn_off
        target:
          entity_id: climate.<your_climate_entity>

    modes:
      - "auto"
      - "heat"
      - "dry"
      - "off"
      - "cool"
      - "fan_only"

    current_temperature_template: "{{ states('sensor.<your_temp_entity>') }}"
    current_humidity_template: "{{ states('sensor.<your_humdi_entity>') }}"
    hvac_mode_template: "{{ states('climate.<your_climate_entity>') }}"

    target_temperature_template: >
      {{ state_attr('climate.<your_climate_entity>', 'temperature') }}

    fan_mode_template: >
      {{ state_attr('climate.<your_climate_entity>', 'fan_mode') }}

    swing_mode_template: >
      {{ state_attr('climate.<your_climate_entity>', 'swing_mode') }}

    set_hvac_mode:
      - service: climate.set_hvac_mode
        target:
          entity_id: climate.<your_climate_entity>
        data:
          hvac_mode: "{{ hvac_mode }}"

    set_temperature:
      - service: climate.set_temperature
        target:
          entity_id: climate.<your_climate_entity>
        data:
          temperature: "{{ temperature }}"

    set_fan_mode:
      - service: climate.set_fan_mode
        target:
          entity_id: climate.<your_climate_entity>
        data:
          fan_mode: "{{ fan_mode }}"

    set_swing_mode:
      - service: climate.set_swing_mode
        target:
          entity_id: climate.<your_climate_entity>
        data:
          swing_mode: "{{ swing_mode }}"
```

## Behavior

With the example above:

- `climate.turn_on` runs `turn_on` and does not force an HVAC mode.
- `climate.turn_off` runs `turn_off`.
- `climate.toggle` uses `turn_on` when the entity is off, and `turn_off` when the entity is on.
- Selecting `cool`, `dry`, `heat`, `auto`, or `fan_only` still runs `set_hvac_mode`.

This mirrors the behavior used by integrations such as Xiaomi climate entities where turning on only sets the device power state and the physical device keeps its previous mode.

## Notes

- To preserve the previous mode, the underlying device or service must support power-on without forcing a fixed mode.
- For IR devices, prefer separate `power_on` and `power_off` commands. A single power toggle command can desynchronize Home Assistant from the real device state.
- If your power-on command includes a fixed mode, for example `cool 25C`, the device will still switch to that fixed mode.
