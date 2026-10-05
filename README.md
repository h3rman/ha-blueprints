# Home Assistant Blueprints

A collection of [Home Assistant](https://www.home-assistant.io/) blueprints.

| Blueprint | Type | Import |
| --- | --- | --- |
| [Badrumsbelysning](#badrumsbelysning) (bathroom lighting) | Automation | [![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fh3rman%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fbadrumsbelysning.yaml) |

## Installation

### One-click import

Click the **Import Blueprint** button next to the blueprint you want. It opens
your own Home Assistant instance through [My Home Assistant](https://my.home-assistant.io/)
with the import dialog pre-filled. Click **Preview** and then **Import blueprint**.

The first time you use a My Home Assistant link you are asked for the URL of
your Home Assistant instance (for example `http://homeassistant.local:8123`).

### Import by URL

1. In Home Assistant, go to **Settings → Automations & scenes → Blueprints**.
2. Click **Import blueprint**.
3. Paste the blueprint's GitHub URL, for example:
   ```
   https://github.com/h3rman/ha-blueprints/blob/main/blueprints/automation/badrumsbelysning.yaml
   ```
4. Click **Preview** and then **Import blueprint**.

### Manual

Copy the YAML file into `<config>/blueprints/automation/h3rman/` in your Home
Assistant configuration directory, then reload automations (or restart Home
Assistant).

### Updating

Blueprints in this repository include a `source_url`, so you can update an
imported blueprint from **Settings → Automations & scenes → Blueprints**: open
the blueprint's ⋮ menu and choose **Re-import blueprint**. Automations created
from the blueprint pick up the changes automatically.

## Blueprints

### Badrumsbelysning

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fh3rman%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fbadrumsbelysning.yaml)

File: [`blueprints/automation/badrumsbelysning.yaml`](blueprints/automation/badrumsbelysning.yaml)
· Requires Home Assistant **2024.10** or newer · UI language: Swedish

Bathroom lighting that turns on with motion or when the door opens. It uses
separate day and night brightness and keeps track of whether someone is still
inside using *wasp in a box* logic: if the door is closed and motion has been
seen after it closed, someone is in there.

| Situation | Behaviour |
| --- | --- |
| Door open | Turns off after a short delay without activity. |
| Door closed, nobody inside | Turns off after a short delay. |
| Door closed, someone inside | Stays on until the door opens. |
| Shower starts behind a closed door (humidity rises) | Counts as someone being inside. |
| Door or motion sensor unavailable | Turns off after a fallback delay. |
| No motion for a very long time | Always turns off eventually (absolute max time). |

At the switch between day and night mode, lights that are already on get the
new brightness. A check runs every 5 minutes as a safety net, so lights still
turn off correctly after restarts, with dead or stuck sensors, or if they were
turned on manually.

#### Requirements

- One or more lights (or a light group).
- A motion, occupancy or presence `binary_sensor`.
- A door or opening `binary_sensor` (`on` = open, `off` = closed).
- A **Toggle** helper (`input_boolean`) used only by this automation. It
  remembers whether someone is inside behind the closed door, even across
  restarts. Create it under **Settings → Devices & services → Helpers →
  Create helper → Toggle**.
- Optional: a humidity sensor in the bathroom, and preferably a reference
  humidity sensor in another room.

#### Inputs

**Enheter (Devices)**

| Input | Description | Default |
| --- | --- | --- |
| Badrumslampa | The light(s) to control. | – |
| Rörelsesensor | Motion / occupancy / presence sensor. | – |
| Dörrsensor | Door / opening sensor. | – |
| Närvarohjälpare | The `input_boolean` helper that stores occupancy. | – |

**Fukt (Humidity, optional)**

| Input | Description | Default |
| --- | --- | --- |
| Fuktsensor i badrummet | Detects a shower starting behind a closed door. | – |
| Referensfuktsensor | Humidity sensor in another room. Recommended: the shower is then detected relative to the rest of the home instead of a fixed limit, so it works equally well in summer and winter. | – |
| Fuktökning som räknas som dusch | Percentage points above the reference sensor that count as a shower. Only used with a reference sensor. | 10 % |
| Fast fuktgräns | Fixed humidity limit. Only used without a reference sensor (or if it is unavailable). | 70 % |

**Ljus (Light)**

| Input | Description | Default |
| --- | --- | --- |
| Nattläge startar | Night mode start. | 23:00 |
| Nattläge slutar | Night mode end. | 06:00 |
| Ljusstyrka (Dag) | Day brightness. | 100 % |
| Ljusstyrka (Natt) | Night brightness. | 10 % |
| Behåll manuell ljusstyrka | If on, motion does not change the brightness of a light that is already on. The day/night switch always resets brightness. | Off |

**Tider (Timing)**

| Input | Description | Default |
| --- | --- | --- |
| Släckfördröjning (Öppen dörr) | Turn-off delay with the door open. | 3 min |
| Släckfördröjning (Stängd dörr, ingen där inne) | Turn-off delay with the door closed and nobody inside, e.g. after someone left and closed the door behind them. A longer value protects someone who goes in, closes the door and sits completely still. | 5 min |
| Rörelsesensorns återställningstid | How long the motion sensor stays `on` after the last motion, plus margin. Must be **longer** than the sensor's own reset time (e.g. Aqara ≈ 60 s, IKEA up to 180 s), otherwise someone who leaves and closes the door is treated as still inside. | 90 s |
| Reservfördröjning (Död sensor) | Turn-off delay when the door or motion sensor is unavailable. | 10 min |
| Absolut maxtid utan rörelse | Always turns off after this long without motion, even if someone is believed to be inside. | 90 min |
| Max tid med rörelse på | If the motion sensor stays `on` this long continuously it is treated as stuck and the light turns off. Useful for PIR sensors; leave at 0 (off) for mmWave / presence sensors, which can legitimately stay `on` for a long time. | 0 (off) |

## Repository layout

```
blueprints/
└── automation/
    └── badrumsbelysning.yaml
```

New blueprints go in `blueprints/<domain>/` (`automation`, `script` or
`template`). Give each one a `source_url` pointing at its file on the `main`
branch, and add it to the table at the top of this README with an import button:

```markdown
[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=<URL-encoded GitHub URL of the blueprint file>)
```
