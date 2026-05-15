# Zone Presence Lights

A Home Assistant blueprint for presence-based light control with lux-based darkness detection, a two-phase timeout, and manual override support.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fthe-baaron%2Fha-zone-presence-lights%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fzone_presence_lights.yaml)

---

## Features

- **Lux-based activation** — uses an illuminance sensor instead of sunset/sunrise, so it adapts to overcast days and indoor light levels
- **Two time windows** — evening scene (05:00–00:00) and night scene (00:00–05:00)
- **Two-phase timeout** — when presence is lost: wait → dim scene → wait → off
- **Presence resumes from dim** — any presence during the dim phase reactivates the full scene and resets the timer
- **Night mode** — immediate off after midnight, no delay, no dim phase (avoids pets keeping lights on)
- **Manual override** — turning lights on or off manually pauses the automation until midnight or noon; no need to disable the automation
- **One blueprint, any room** — create one automation per zone with its own scenes, sensor, and timing

---

## Requirements

### Helpers

Create one **Toggle** helper per zone in **Settings → Helpers**:

| Helper | Example name | Example entity ID |
|---|---|---|
| Toggle (input_boolean) | Living room manual override | `input_boolean.living_room_manual_override` |

### Devices

- A **presence sensor** (binary_sensor)
- An **illuminance sensor** (lux) — many presence sensors include one
- One or more **lights**

### Scenes

Create scenes for each zone in **Settings → Scenes** or in `scenes.yaml`. You need:

| Scene | Purpose | Suggested brightness |
|---|---|---|
| Evening | Active presence, evening hours | 30–60% |
| Night *(optional)* | Active presence, after midnight | 1–5% |
| Dim *(optional)* | Fading out after no presence | 10–20% |

See [`examples/scenes.yaml`](examples/scenes.yaml) for a starting point.

---

## Installation

### Via HACS (recommended)

1. Open HACS in your Home Assistant instance
2. Go to **Automations** → click the three dots → **Custom repositories**
3. Add `the-baaron/ha-zone-presence-lights` with category **Blueprint**
4. Find **Zone Presence Lights** and download it

### Manual

1. Copy [`blueprints/automation/zone_presence_lights.yaml`](blueprints/automation/zone_presence_lights.yaml) into your HA config at:
   ```
   config/blueprints/automation/zone_presence_lights.yaml
   ```
2. Reload blueprints in **Developer Tools → YAML → Blueprints**

---

## Setup

1. Create the required helper (see above)
2. Create your scenes
3. Go to **Settings → Automations → Create automation → Use a blueprint**
4. Select **Zone Presence Lights** and fill in the inputs

See [`examples/automations.yaml`](examples/automations.yaml) for a full example.

---

## How it works

```
Presence detected + dark (lux ≤ threshold)
  └── 05:00–00:00 → Evening scene
  └── 00:00–05:00 → Night scene (or nothing if not set)

Presence lost
  └── 05:00–00:00 → wait off-delay → dim scene → wait dim timeout → off
  └── 00:00–05:00 → immediately off

Presence returns during any timer → scene reactivates, timer resets

Manual on or off at any time → automation pauses until 12:00 or 00:00
```

### Darkness threshold

The lux threshold determines when the automation activates. A good starting point is **50 lx** — roughly the light level indoors at dusk. Adjust based on your sensor placement and preference.

### Fade time

All transitions (on and off) use the configured fade time. Set to `0` for instant changes.

---

## Blueprint inputs

| Input | Required | Default | Description |
|---|---|---|---|
| Presence sensor | Yes | — | binary_sensor detecting occupancy |
| Zone lights | Yes | — | Light entities to monitor and control |
| Illuminance sensor | Yes | — | Lux sensor for darkness detection |
| Darkness threshold | Yes | 50 lx | Activate when lux is at or below this |
| Evening scene | Yes | — | Scene for 05:00–00:00 |
| Night scene | No | — | Scene for 00:00–05:00 |
| Dim scene | No | — | Scene activated after off-delay expires |
| Dim timeout | No | 60 min | How long to hold the dim scene |
| Manual override | No | — | Toggle helper for manual control |
| Fade time | No | 60 s | Transition time for all changes |
| Off delay (evening) | No | 30 min | Wait after presence lost before dimming |

---

## License

MIT
