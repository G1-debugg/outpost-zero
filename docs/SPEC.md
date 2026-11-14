# Outpost Zero: Game Spec (single source of truth)

## 1. Pitch
You are a trainee engineer at the Junior Astronaut Academy. A simulator replays real space-mission emergencies as a **living pixel-art base you walk around in**. Keep four crew alive for one shift by running around the base: flip breakers, haul ice, fix leaks, sweep dust, shelter from radiation. At the end, "What really happened" shows the real mission next to your choices.

Look: GBA-era top-down pixel art (16x16 tiles, 240x160, chunky dialogue boxes). No battle menus, no collecting, no badges. Closer to Overcooked tasks with Oxygen Not Included systems behind the walls.

## 2. Controls (one-button rule)
- Move: arrows/WASD, or touch D-pad. Grid steps of 16 px, smooth tween.
- **A** (Z / Space / big touch button): do the thing in front of you. **B** (X / small touch button): scanner. **Menu** (Esc): pause, Logbook, Data Sources, Settings.

## 3. Verbs (each teaches one trade-off)
| Verb | How | Teaches |
|---|---|---|
| Flip breakers | A on a room's breaker box toggles the room | Power budgeting |
| Carry | Hold one item (ice, regolith bag, spare part) and drop it on a machine | Making air and water from local resources |
| Repair | 3 short one-button mini-games (wires, bolt, leak). Retry is free. | Fixing under pressure |
| EVA | Airlock cycle, suit with own air timer, sweep solar panels, collect ice | Air and dust cost of going outside |
| Build shielding | Haul regolith bags to a wall marker, stack with A | Radiation shielding |
| Shelter | During a radiation alarm get everyone to the shelter | Timing and protection |
| Scan and talk | B scans an object for a real fact. A talks to crew. | Real science |

## 4. Resources (exactly three, also shown physically in the world)
- **Power** (battery 0-100): solar adds, rooms and machines subtract. Low power: lights flicker, rooms go dark, heaters then oxygen maker stop.
- **Air** (0-100) and CO2: crew use air; oxygen maker (needs power and water) adds air. Low air: alarm, crew slow down, speech bubbles say "I feel dizzy."
- **Shield** (per room value): lowers radiation dose. Crew and player each carry a **dose** number shown on a badge.
No free choices: oxygen maker needs power; heaters need power; EVA costs air and adds a little dose; shielding costs crew time and hauling.

**Delayed cost reveal:** every cost that lands later shows a one-line replay ("You turned off the greenhouse. The plants got too cold.").

## 5. Base (one layout, reused with palette swaps)
Rooms: Power Bay, Life Support (oxygen maker, melter, CO2 scrubber), Greenhouse, Habitat, Comms Hub, Lab, Storm Shelter, Airlock. Outside: solar yard with 6 panels, ice patches, rover bay.
```
 [Solar yard, outside]
        |
 [Power Bay]--[Life Support]--[Greenhouse]
        |            |
   [Habitat]----[Comms Hub]--[Lab]
        |
 [Storm Shelter]   [Airlock]--> outside
```
Map file: src/content/maps/base.txt (ASCII), legend in legend.json. Tile legend must include: floor, wall, door, breaker, machine, terminal, pickup, deposit, repair spot, airlock, scan spot, crew post, shelter floor, panel, ice, dust. Rooms are rectangles with `powerDraw` and `shield` values from balance.json.

## 6. Time
One shift = 16 game hours. 1 game hour = 15 real seconds (config). Total about 4 minutes of play per mission plus 1 minute briefing and 2 minutes debrief. Pause is always available.

## 7. Crew (4, simple routines)
Defined in src/content/crew.json. They walk between rooms on a schedule, show a speech-bubble need (warm, light, air, help), and ask the player for help. Helping one often means ignoring another.

## 8. Missions
Each mission = a JSON timeline in src/content/missions/*.json plus a research file. Real-mission framing: "The simulator drops you in when things went wrong."

### Mission 1: The Great Dust Storm (Mars). Research: opportunity.md
Honest framing: Opportunity was a solar-powered rover, not a base. The sim asks "what if a Mars outpost were in the same storm?" The debrief tells what really happened to the rover.
| Hour | Event |
|---|---|
| 0 | Tutorial: flip a breaker, carry ice, fix a wire |
| 3 | Delayed message from Earth reaches Comms Hub: dust storm spotted (delay = today's real Earth-Mars light time, scaled) |
| 5 | Sky turns orange. Solar x0.5 |
| 8 | Solar x0.1. Battery drains. Choose which rooms to switch off |
| 10 | Crew requests: lab heat vs greenhouse light. Cannot have both |
| 13 | Storm fades. Panels covered in dust. EVA to sweep them |
| 16 | Debrief |

### Mission 2: The Storm That Missed (Moon, south pole). Research: aug1972.md
Real-data mission. Today's space weather sets the radiation event (see section 9). Warning countdown, alarm, crew run to shelter, shielding you built earlier lowers dose. Debrief compares your crew dose with a moonwalker in the 1972 storm and with the Apollo hull.

### Mission 3 (stretch): The Long Night (Moon). Research: lunar-night.md
Zero solar power for the whole shift. Heat vs air vs light. Debrief: Chang'e-4 (heater units) vs Chandrayaan-3 (no heater units). Radiation tile from RadLab Chang'e-4 lander data if the API works, else labelled ARCHIVE.

## 9. Real data (honesty rules)
Every data tile shows **LIVE** or **ARCHIVE (date)**. Fetch through /functions proxy where needed, cache 3 hours, fall back to src/content/archive/*.json (made by `npm run snapshot` from real responses, never typed by hand).

| Source | Use | Endpoint (agent must confirm the real response shape on first call) |
|---|---|---|
| NOAA SWPC scales | Today's Storm Level | https://services.swpc.noaa.gov/products/noaa-scales.json |
| NOAA SWPC protons | Radiation backup and detail | https://services.swpc.noaa.gov/json/goes/primary/integral-protons-1-day.json |
| NOAA SWPC Kp | Geomagnetic backup | https://services.swpc.noaa.gov/json/planetary_k_index_1m.json |
| NASA DONKI | Recent flares/SEP/storms for the Field Guide and Mission 2 flavour | https://api.nasa.gov/DONKI/{FLR,SEP,GST,notifications}?startDate=&endDate=&api_key= (default window 30 days) |
| JPL Horizons | Earth-Mars one-way light time for Mission 1 | https://ssd.jpl.nasa.gov/api/horizons.api (target 499, observer from 500@399, one-way light-time quantity; confirm the quantity number in the docs) |
| OSDR RadLab | Real dose-rate numbers for the Radiation Passport and Mission 3 | https://visualization.osdr.nasa.gov/radlab/api/ (units: microgray per hour; includes ISS instruments and the Chang'e-4 lander) |

**Storm Level rule** (tunable in balance.json): Storm if S-scale >= 1 or G-scale >= 3 or R-scale >= 3. Watch if G >= 2 or R >= 2 or Kp >= 5. Otherwise Calm.
**Calm-day problem:** on a calm day Mission 2 would be boring. So Calm gives a "training storm" (small, but real-looking), and the Hub also offers **Replay a real storm** (ARCHIVE, labelled with its date, for example the May 2024 storm) so every class gets a real challenge.

## 10. Learning layer
- Scanner and Logbook: 2-sentence real fact, source label, optional NASA photo link (no hot-linking at runtime; photos only if bundled and credited).
- Glossary: words in {{double braces}} open a plain-language pop-up. Never blocks play.
- **Radiation Passport**: crew dose shown next to everyday comparisons, from src/content/radiation.json (with sources and conflicts flagged).
- Debrief: "What really happened" next to a timeline of the player's key decisions, 4 scores (Safety, Science, Resilience, Curiosity) with rules in balance.json, one of 8 endings, soft words only, Retry and Continue, and a printable one-page Mission Report (browser print).
- Data Sources, Credits and "What's real" screens, generated from sources.json.

## 11. Look and sound (zero downloads)
- Palette: src/art/palette.json (16 colours). Mars = orange-red ground tint, Moon = grey-blue with star sky.
- Sprites: pixel-string definitions compiled at boot. Player and 4 crew share one body template with different colours.
- Rooms go dark with an alpha overlay. Flicker is an alpha jitter. Alarm lights are animated tiles. Dust is a particle emitter.
- Audio: WebAudio synth (blips, hum, alarm, footsteps) and a tiny looping 4-bar chiptune. Mute button.
- Font: Pixelify Sans via @fontsource. 

## 12. Quality bars
- First shift playable without reading anything except tooltips.
- Mobile portrait and landscape, touch and keyboard. Initial load under 5 MB. Works offline after first visit (PWA).
- Reading level: max 12 words per sentence.
- Balance: a scripted bot suite proves (a) idle play fails softly, (b) a sensible strategy reaches partial success, (c) a good strategy reaches full success. See scripts/balance-bots.ts.

## 13. Scope tiers
- **MVP:** M0-M5 (Mission 1 fully playable, Debrief, Data Sources, deployed).
- **Should:** M6-M9 (shelter, live space weather, Mission 2, Logbook, glossary, audio, save, PWA).
- **Stretch:** M10 Mission 3, M13 art upgrade, M14 Hindi and Marathi.
