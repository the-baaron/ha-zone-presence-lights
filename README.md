# Zone Presence Lights

A Home Assistant blueprint that turns lights on when presence is detected and it's dark, and off when you leave. Uses a lux sensor instead of sunset/sunrise — works on overcast days and responds to indoor light changes.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fthe-baaron%2Fha-zone-presence-lights%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fzone_presence_lights.yaml)

---

## Features

- **Lux-based activation** — works with any illuminance sensor, indoor or outdoor
- **Activates in both directions** — turns on when you enter a dark room, and also when it gets dark while you're already there
- **Two time windows** — default mode (06:00–00:00) with delays and fading; night mode (00:00–06:00) with instant on/off and no timers
- **Two-phase timeout** — presence lost → off-delay → dim scene → dim timeout → off
- **Presence resumes from dim** — returning during the dim phase reactivates the full scene and resets the timer
- **Manual override** — manually turning lights on or off sets *Forced on* or *Forced off* until midnight or noon
- **One blueprint, any room** — each automation instance has its own sensor, scenes, and timing

---

## Requirements

### Helpers

Create one **Select** helper per zone in **Settings → Helpers**:

| Helper type | Options |
|---|---|
| Select (`input_select`) | `Auto`, `Forced on`, `Forced off` |

See [`examples/helpers.yaml`](examples/helpers.yaml) for the `configuration.yaml` snippet.

### Devices

- A **presence sensor** (`binary_sensor`)
- An **illuminance sensor** — many presence sensors include one; an outdoor sensor also works
- One or more **lights**

### Scenes

| Scene | Purpose | Suggested brightness |
|---|---|---|
| Default | Active presence, 06:00–00:00 | 30–60% |
| Night *(optional)* | Active presence, 00:00–06:00 | 1–5% |
| Dim *(optional)* | Fading out after presence is lost | 10–20% |

See [`examples/scenes.yaml`](examples/scenes.yaml) for a starting point.

---

## Installation

### Via HACS

1. Go to **HACS → Automations** → three dots → **Custom repositories**
2. Add `the-baaron/ha-zone-presence-lights` with category **Blueprint**
3. Download **Zone Presence Lights**

### Manual

Copy [`blueprints/automation/zone_presence_lights.yaml`](blueprints/automation/zone_presence_lights.yaml) into:
```
config/blueprints/automation/zone_presence_lights.yaml
```
Then reload blueprints in **Developer Tools → YAML → Blueprints**.

---

## Setup

1. Create the select helper (see above)
2. Create your scenes
3. Go to **Settings → Automations → New automation → Use a blueprint**
4. Select **Zone Presence Lights** and fill in the inputs

See [`examples/automations.yaml`](examples/automations.yaml) for a full example.

---

## How it works

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

### Darkness threshold

Tip: check your sensor's current value in **Developer Tools → States** before setting this. Indoor sensors typically read 5–50 lx at dusk; outdoor sensors 100–1,000 lx.

### Night mode

No fade, no delays. Lights snap on the moment presence is detected and off the moment it's gone. Useful for avoiding long timeouts from pets.

---

## Blueprint inputs

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
| Timing | Off delay | No | 30 min | Wait after presence lost before dimming/off |
| Timing | Dim timeout | No | 60 min | How long to hold the dim scene |
| Override | Override helper | No | — | `input_select` for manual control state |

---

## License

MIT
