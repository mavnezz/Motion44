# Motion44

A slim Home Assistant **blueprint** for motion-triggered lighting, with separate
evening and night profiles, lux/sun-based dark detection, and an optional
bypass via an `input_boolean` helper.

Built as a stripped-down alternative to the excellent but very large
[Blackshome `sensor-light`](https://gist.github.com/Blackshome/6edfec0ff6a25c5da0d07b88dc908238)
blueprint — covers the 80% of cases you actually need without 10,000 lines of
YAML.

## Features

- One or more motion sensors → turn lights on, turn off after a configurable delay
- **Dark detection** as the gate — won't fire when it's already bright
  - Sun-based (sun below horizon)
  - Lux-based (illuminance sensor below threshold)
  - Both can be combined (AND)
- **Two profiles** with independent brightness and color temperature
  - **Evening profile**: used outside the night window
  - **Night profile**: used inside the configured time window (e.g. for a
    dimmer, warmer toilet light at night)
- **Optional bypass** via an `input_boolean` helper
  - While the helper is `on`, the automation neither turns lights on nor off
  - Toggle the helper however you want — dashboard switch, voice, or a tiny
    side-automation from your remote buttons

## Quick install

In Home Assistant: **Settings → Automations & Scenes → Blueprints → Import Blueprint**, paste:

```
https://github.com/mavnezz/Motion44/blob/main/blueprints/automation/motion44/motion_light.yaml
```

Click **Preview** → **Import Blueprint**. Then create one automation per room.

## Manual install

Copy `blueprints/automation/motion44/motion_light.yaml` to
`<config>/blueprints/automation/motion44/motion_light.yaml` on your Home
Assistant host. Restart or reload blueprints.

## Configuration

### Motion & lights

| Setting | Description |
|---|---|
| `motion_sensors` | One or more `binary_sensor` entities that report motion. |
| `lights` | Target — `light` and/or `switch` entities to control. |
| `no_motion_wait` | Seconds after the last motion event before turning lights off (default: `120`). |

### Dark detection (gate)

The light only turns on when these conditions allow it. Both checks are
optional and AND-combined when both are enabled.

| Setting | Description |
|---|---|
| `use_sun` | If on, only fire when `sun.sun` is `below_horizon`. |
| `use_illuminance` | If on, only fire when the lux sensor reports below the threshold. |
| `illuminance_sensor` | A `sensor` with `device_class: illuminance`. |
| `lux_threshold` | Fire only when current lux is **below** this value (default: `50`). |

### Brightness / color

Toggles apply to **both** profiles. Per-profile values are configured below.
Only relevant for `light` entities that support these features — switches
ignore them.

| Setting | Description |
|---|---|
| `use_brightness` | Apply the brightness percentage when turning on. |
| `use_color_temp` | Apply the color temperature when turning on. |

### Evening profile (default when dark)

Used outside the night window (or always, if `use_night_profile` is off).

| Setting | Description |
|---|---|
| `brightness_pct_evening` | Brightness in percent (default: `100`). |
| `color_temp_kelvin_evening` | Color temperature in Kelvin (default: `3000`). |

### Night profile (inside time window)

| Setting | Description |
|---|---|
| `use_night_profile` | If on, switch to night settings inside the configured window. |
| `night_window_start` | Start time (default: `22:00:00`). |
| `night_window_end` | End time (default: `06:00:00`). Wraparound across midnight is supported. |
| `brightness_pct_night` | Brightness in percent (default: `20`). |
| `color_temp_kelvin_night` | Color temperature in Kelvin (default: `2200`). |

### Bypass

Pause/resume the automation by toggling an `input_boolean` helper. The
blueprint only **reads** this helper — it does not toggle it itself. That
keeps the blueprint simple and lets you wire the toggle however suits you.

| Setting | Description |
|---|---|
| `use_bypass` | Master toggle for the bypass feature. |
| `bypass_helper` | An `input_boolean` you create in HA. While `on`, motion is ignored. |

Common ways to toggle the helper:

- A switch on your dashboard
- A voice command via Assist
- A tiny separate automation: `binary_sensor` button press → `input_boolean.toggle`

## Example: hallway toilet

- **Motion sensor**: motion sensor in the hallway
- **Lights**: tunable-white ceiling light
- **Dark detection**: `use_sun` on (don't fire during daytime)
- **Evening profile**: 100% at 3000 K (normal walking-around light)
- **Night profile**: on, window `22:00`–`06:00`, 20% at 2200 K (gentle toilet light)
- **Bypass**: optional — toggle a `Motion44 Hallway Bypass` helper from a Hue Remote during cleaning, parties, etc.

## Setup checklist for the bypass

1. **Settings → Devices & services → Helpers → Create Helper → Toggle**, give it
   a unique name like `Motion44 Hallway Bypass`.
2. In the blueprint instance, enable `use_bypass` and point `bypass_helper` to that helper.
3. Toggle the helper from wherever you like (dashboard switch, voice, side-automation).

## What is intentionally NOT included

The original Blackshome blueprint covers many more scenarios. Motion44 leaves
out the following on purpose to keep things simple:

- Scenes/scripts as targets (use a separate automation)
- Dynamic lighting curves (lux → brightness mapping)
- Night-glow ambient light when no motion
- Manual override based on light state (use the bypass instead)
- Device tracker / presence checks
- HA restart safeguards (modern HA handles this gracefully)
- Bathroom humidity integration

If any of these are critical for your use case, feel free to fork or open an
issue.

## Compatibility

- Tested with Home Assistant **2024.6+** (uses the modern `triggers:` /
  `actions:` keys and `selector: time`)
- Works with any motion `binary_sensor` (Zigbee, Z-Wave, ESPHome, etc.)
- Bypass requires only an `input_boolean` helper — toggle it however you want

## Credits

Inspired by [Blackshome's `sensor-light`](https://gist.github.com/Blackshome/6edfec0ff6a25c5da0d07b88dc908238).

## License

MIT
