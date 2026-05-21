# HA Zone Presence Lights

A collection of Home Assistant blueprints for presence-based lighting and automation.

---

## Blueprints

### Zone Presence Lights

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fthe-baaron%2Fha-zone-presence-lights%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fzone_presence_lights.yaml)

Turns lights on when presence is detected and it's dark, off when you leave. Uses a lux sensor instead of sunset/sunrise — works on overcast days and responds to indoor light changes.

**Features:**
- **Lux-based activation** — works with any illuminance sensor, indoor or outdoor
- **Activates in both directions** — turns on when you enter a dark room, and also when it gets dark while you're already there
- **Two time windows** — default mode (06:00–00:00) with delays and fading; night mode (00:00–06:00) with instant on/off and no timers
- **Two-phase timeout** — presence lost → off-delay → dim scene → dim timeout → off
- **Presence resumes from dim** — returning during the dim phase reactivates the full scene
- **Manual override** — manually turning lights on/off sets *Forced on* or *Forced off* until midnight or noon

[Full docs →](docs/zone_presence_lights.md)

---

### Door Light

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fthe-baaron%2Fha-zone-presence-lights%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fdoor_light.yaml)

Simple contact sensor → light blueprint. Light turns on when the door/window opens, off when it closes.

---

### Low Battery Alert

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fthe-baaron%2Fha-zone-presence-lights%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Flow_battery_alert.yaml)

Sends a notification when a battery sensor drops below a configurable threshold. Works with any `battery` device class sensor.

---

### Adaptive Brightness

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fthe-baaron%2Fha-zone-presence-lights%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fadaptive_brightness.yaml)

Applies a scene based on the time of day whenever a light turns on. Supports up to four configurable time windows (morning, afternoon, evening, night) — all optional.

---

### Away Mode

[![Import blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fthe-baaron%2Fha-zone-presence-lights%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Faway_mode.yaml)

Tracks multiple presence sensors. Triggers an away scene when all sensors have been off for a configurable delay, and a home scene the moment any sensor activates.

---

## Installation

### Via HACS

1. Go to **HACS → Automations** → three dots → **Custom repositories**
2. Add `the-baaron/ha-zone-presence-lights` with category **Blueprint**
3. Download the blueprint(s) you want

### Manual

Copy any `.yaml` file from `blueprints/automation/` into:
```
config/blueprints/automation/
```
Then reload blueprints in **Developer Tools → YAML → Blueprints**.

---

## Zone Presence Lights — Full Reference

### Requirements

**Helpers** — create one Select helper per zone in **Settings → Helpers**:

| Helper type | Options |
|---|---|
| Select (`input_select`) | `Auto`, `Forced on`, `Forced off` |

See [`examples/helpers.yaml`](examples/helpers.yaml) for the `configuration.yaml` snippet.

**Devices:**
- A presence sensor (`binary_sensor`)
- An illuminance sensor — many presence sensors include one; an outdoor sensor also works
- One or more lights

**Scenes:**

| Scene | Purpose | Suggested brightness |
|---|---|---|
| Default | Active presence, 06:00–00:00 | 30–60% |
| Night *(optional)* | Active presence, 00:00–06:00 | 1–5% |
| Dim *(optional)* | Fading out after presence is lost | 10–20% |

See [`examples/scenes.yaml`](examples/scenes.yaml) for a starting point.

### How it works

```
Presence detected + dark (lux ≤ threshold)
  └── 06:00–00:00 → Default scene (with fade)
  └── 00:00–06:00 → Night scene, or nothing if not configured

Lux drops below threshold while presence is already active
  └── Same activation as above

Lux rises above threshold → lights turn off

Presence lost + lights on
  └── 06:00–00:00 → wait off-delay → dim scene (optional) → wait dim timeout → off
  └── 00:00–06:00 → immediately off, no delay

Presence returns during off-delay or dim phase → full scene reactivates, timer resets

Manual on  → Forced on  — automation paused until 12:00 or 00:00
Manual off → Forced off — automation paused until 12:00 or 00:00
```

**Darkness threshold tip:** check your sensor's current value in **Developer Tools → States** before setting this. Indoor sensors typically read 5–50 lx at dusk; outdoor sensors 100–1,000 lx.

**Night mode:** no fade, no delays. Lights snap on when presence is detected and off when it's gone.

### Blueprint inputs

| Section | Input | Required | Default | Description |
|---|---|---|---|---|
| Zone | Presence sensor | Yes | — | `binary_sensor` detecting occupancy |
| Zone | Lights | Yes | — | Light entities to control |
| Light level | Brightness sensor | Yes | — | Illuminance sensor (indoor or outdoor) |
| Light level | Darkness threshold | Yes | 50 lx | Lights activate at or below this value |
| Scenes | Default scene | Yes | — | Scene for 06:00–00:00 |
| Scenes | Night scene | No | — | Scene for 00:00–06:00 |
| Scenes | Dim scene | No | — | Scene activated after off-delay expires |
| Timing | Fade time | No | 60 s | Transition duration (06:00–00:00 only) |
| Timing | Night fade time | No | 0 s | Transition duration (00:00–06:00) |
| Timing | Off delay | No | 30 min | Wait after presence lost before dimming/off |
| Timing | Dim timeout | No | 60 min | How long to hold the dim scene |
| Override | Override helper | No | — | `input_select` for manual control state |

---

## License

MIT
