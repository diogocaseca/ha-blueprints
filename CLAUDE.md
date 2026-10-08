# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A flat collection of Home Assistant **automation blueprints** (`domain: automation`), one YAML file per blueprint at the repo root. There is no build, test suite, or tooling — files are consumed directly by Home Assistant, typically imported via their raw GitHub URL (`github.com/diogocaseca/ha-blueprints`). Validation happens by importing/reloading the blueprint in a Home Assistant instance.

The YAML uses Home Assistant's custom `!input` tag, so generic YAML parsers/linters will reject it unless they're configured to ignore unknown tags.

## Blueprint families

**Zigbee2MQTT device remotes** (`ERS-10TZBVK-AA.yaml`, `ESW-0ZAA-EU.yaml`), named after the device model:
- Inputs `mqtt_entity` (Z2M friendly name) and `base_topic` (default `zigbee2mqtt`) are exposed via `trigger_variables` so the MQTT trigger topic can be templated as `{{ base_topic ~ '/' ~ mqtt_entity ~ '/action' }}`.
- The action payload is dispatched through a single `choose:` block, one branch per device action string, each running an `action`-selector input (`default: []` so every action is optional).
- `ERS-10TZBVK-AA` (smart knob) additionally triggers on a Z2M `select` entity (`state_entity`) for the operation mode (`event`/`command`). It derives `platform` (from `trigger.id`) and `payload` from the trigger so MQTT action strings and state changes share one `choose:`; branches check both `platform` and `payload`. Its `mode: queued` (max 50) is deliberate so rapid rotation events aren't dropped. The comment block in the file lists the device's payloads per mode (source: the manuals.plus link there).

**Schedule helpers** (`schedule-action.yaml`, `schedule-switch.yaml`):
- Trigger on a `schedule` entity changing to `on`/`off`, compute `payload` from `trigger.to_state` (falling back to `states(entity_schedule)`), then branch with `if/then/else`.
- `schedule-switch` drives a `switch` entity and runs optional `extra_actions`; `schedule-action` runs user-supplied on/off action sequences.

## Conventions

- Use the current HA syntax (`triggers:` with `- trigger: <platform>`, `actions:`, and `action:` for service calls). The legacy keys (`trigger:` / `platform:` / top-level `action:` / `service:`) cause deprecation repairs in HA. To tell triggers apart, give each one an `id` and branch on `trigger.id`, not on `trigger.platform`.
- Input names follow `<group>_<event>` (e.g. `btn_1_double`, `event_rotate_left`, `command_brightness_step_up`), matching the device's Z2M action payload where possible.
