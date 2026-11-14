# Milestones and prompts

How to use: paste ONE prompt into a new agent chat. Use Antigravity for M0-M10. Use Copilot agent mode for the Fix prompt. Do not start the next milestone until `npm run check` is green and you have committed.

Every prompt begins with the same header. Copy it with the prompt.

HEADER (paste first, every time):
> Read AGENTS.md, docs/SPEC.md and the research files. Work autonomously. Write code AND tests, run `npm run check`, look at screenshots of the running game, and fix problems yourself until everything is green. Do not ask me questions unless blocked. End with the short report AGENTS.md requires.

---
## M0. Scaffold, checks, deploy pipeline
> Do milestone M0. Create a Vite + TypeScript (strict) + Phaser 4 project in this repo (keep the existing files). Pin exact versions. Copy the skills/ folder from the phaserjs/phaser GitHub repo into docs/phaser-skills/ and read the parts you need. Scenes: Boot, Title, Hub, Base, Debrief (empty for now). 240x160 internal resolution with integer scaling that stays crisp on any screen, and a phone-friendly layout. Title screen with the game name and Start. Add npm scripts: dev, build, typecheck, lint, lint:text (fails on any kid-facing sentence over 12 words in src/content/text/*.json), test (Vitest), smoke (Playwright, desktop and 390x844, loads the game, presses Start, saves a screenshot to docs/screens), check (runs all of them and prints ALL GREEN). Add vite-plugin-pwa with an empty offline cache for now. Add a /functions folder with a health-check function for Cloudflare Pages. Update .gitignore. Tell me the exact commands to run.
**You should see:** `npm run check` ends with ALL GREEN. `npm run dev` shows a title screen. After you connect Cloudflare (START_HERE Part 4), a public link works.

## M1. Art engine and walking
> Do milestone M1. Build src/art: a compiler that turns palette-indexed pixel strings (using src/art/palette.json) into Phaser textures at boot. Draw, in code: floor, wall, door, breaker box, terminal, machine (3 states), crate, solar panel (clean and dusty), ice patch, airlock door, bed, plant, lab bench, and a body template for the player and 4 crew in different colours (4 directions, 2 walk frames). Write src/content/maps/base.txt and legend.json for the base in SPEC section 5 (about 60x40 tiles, 8x6 rooms, 2-wide corridors, 1-wide doors) plus an outside map. Build a Vitest map validator (all rooms reachable, every required object exists exactly where the spec needs it). Base scene: draw the map, grid-based movement with smooth tween, wall collision, camera that follows and stays in the map, touch D-pad plus one big A button (only on touch devices, portrait and landscape, no page scroll or zoom). Add a debug page /?art that shows every sprite on a grid so I can judge the art. Make the art charming and readable: strong outlines, consistent light from the top-left, 3 shades per surface.
**You should see:** a pixel base you can walk around, on laptop and phone. Open `/?art` to review sprites. If the art looks rough, tell the agent exactly what looks wrong ("the walls look flat") and repeat the Fix prompt.

## M2. Simulation, breakers, darkness, HUD
> Do milestone M2. Build src/sim (no Phaser imports): SimClock (pausable, speed from balance.json), BaseState (battery, air, co2, per-room on/off and draw, per-room shield), per-hour updates exactly as in docs/SPEC.md section 4 using src/content/balance.json. Unit-test every rule. Connect to the Base scene: A on a breaker toggles its room with a click sound hook and animation. A room that is off gets a dark overlay (alpha tween), flickers when battery is low, and its machines stop. HUD with three icons and bars (Power, Air, Shield) and a clock. Power Bay terminal shows a battery gauge and a watts-in vs watts-out bar graph. Backtick key toggles a debug overlay with live sim numbers.

## M3. Machines, carrying, crew
> Do milestone M3. Carrying: the player holds one item (ice, regolith bag, spare part) drawn above their head. Chain: ice to Melter (power) to water to Oxygen Maker (power) to Air, plus the CO2 scrubber. Machines show states (running, idle, no power, no input) and a progress bar. Gentle hint on wrong drops. Crew: 4 NPCs from src/content/crew.json walk between rooms on schedule (simple pathfinding), consume air, show a speech-bubble need icon when unmet, and give a need-dependent line when talked to. Tests: sim chain math, pathfinding reaches every room, no crew gets stuck in 10 simulated shifts.

## M4. Repairs and EVA
> Do milestone M4. Three one-button repair mini-games under 10 seconds each, retryable: wires (match colours), bolt (stop marker in green zone), leak (hold A to fill). An ignored repair keeps a visible effect (leak loses Air, short circuit adds power drain). Airlock and outside: A on the inside door starts a 3-second pressure cycle that costs Air, then outside map with a suit air bar draining, a low-air warning, sweeping a panel (hold A) restores solar output, collecting ice, a dust-off step at the airlock, dose added per minute outside. If suit air runs out the player is returned to the airlock with "mission paused for rescue" and a time penalty, never a hard fail. Tests for each rule.

## M5. Events, Comms Hub, Mission 1, Debrief (FIRST PLAYABLE GAME)
> Do milestone M5. EventDirector reading a mission timeline JSON (message, weather, alarm, breakdown, crewRequest, radiation). Comms Hub terminal with queued timestamped messages and a blinking HUD icon. Sky tint and solar curve changes. Write src/content/missions/opportunity.json from SPEC section 8 and src/content/research/opportunity.md only (kid text in src/content/text/en.json, max 12 words per sentence; use TODO(fact) for anything missing). Add the Hub scene (choose a mission, Data Sources button). Debrief scene: "What really happened" next to a timeline of the player's key decisions recorded by the sim, 4 scores, one of 8 endings from JSON rules, soft wording only, Retry, printable Mission Report. Also write scripts/balance-bots.ts: three scripted players (idle, sensible, expert) that play Mission 1 headless; assert idle fails softly, sensible gets partial success, expert gets full success; adjust balance.json until true. Add a test that no mission action has zero cost.
**You should see:** a complete 5-minute Mission 1 from Title to Debrief. **Stop here. Do the child playtest.** Take notes on where the child got confused and paste them into the Fix prompt.

## M6. Radiation, shelter, live space weather, Mission 2
> Do milestone M6. Radiation: warning countdown, loud alarm, radiation levels 0-3 from balance.json, dose per person each hour reduced by the room's shield, crew auto-run to the Storm Shelter, crowding uses more Air, badges show dose. Build-shielding task: carry regolith bags to a wall marker and stack. SpaceWeatherService: first, call each live endpoint in SPEC section 9 from the terminal, print the real response shape, and write what you found to docs/data-notes.md. Then implement the service through a Cloudflare Pages Function (NASA key from env NASA_API_KEY, never in client code), with NOAA scales as primary, protons/Kp as backup, DONKI for recent events, 3-hour cache, and the archive fallback. Create scripts/snapshot.ts (`npm run snapshot`) that saves real responses with timestamps into src/content/archive/; run it now. Return {level, source, isLive, timestamp, detail}; apply the Storm Level rule from balance.json. Hub tile shows LIVE or ARCHIVE (date). Add "Replay a real storm" (ARCHIVE) for calm days, a debug key to force each level, and Mission 2 JSON (src/content/missions/aug1972.json) using only aug1972.md. Banner at start: "Today the Sun is [level], so your storm is [size]." Moon palette and star sky. Extend the balance bots to Mission 2 at all three levels.

## M7. Learning layer
> Do milestone M7. Scanner (B button) logs a real fact from the research sheets into a Logbook (pause menu) with source label. {{glossary}} pop-ups from src/content/glossary.json (never block play). Radiation Passport screen from src/content/radiation.json (use the status flags; show conflicting values as "about"). Data Sources, Credits and "What's real" screens generated from src/content/sources.json, reachable from Title, Hub and the pause menu. A test fails the build if any data tile lacks a LIVE or ARCHIVE label.

## M8. Sound and feel
> Do milestone M8. WebAudio synthesiser in src/audio: UI blips, breaker click, machine hum that fades with distance, footsteps (different inside and outside), alarm, airlock hiss, a 4-bar looping chiptune per planet. Mute button and Settings toggle. Juice without changing logic: screen shake on alarms, flicker, floating dust particles outside, animated warning lights, smooth day-tint. Keep 60 fps on a mid-range phone (measure with a Playwright performance check and report the number).

## M9. Save, settings, offline, accessibility, performance
> Do milestone M9. Autosave and Continue (localStorage, versioned, safe reset if corrupt). Settings: sound, text speed, large text, reduce flashing, colour-safe alarm (not red/green only). Full offline PWA (the whole game plays with no network; space-weather tile falls back to ARCHIVE and says so). Initial load under 5 MB (report the real size). Keyboard-only and touch-only Playwright tests for the full Mission 1. Run an accessibility pass and fix what you find.
**You should see:** airplane-mode play works after one online visit. **Human jobs now:** child playtest 2 and the demo video.

## M10. (Stretch) Mission 3, The Long Night
> Do milestone M10. Add src/content/missions/lunar-night.json from lunar-night.md: zero solar power, heat vs air vs light. Add a heat value per room (balance.json). Debrief compares Chang'e-4 and Chandrayaan-3 neutrally. Try the RadLab API for Chang'e-4 lander dose rates; if it works show them LIVE-from-archive with the data's real date, else use a snapshot. Extend balance bots.

## M11. Whole-repo QA
> Do milestone M11. Do NOT add features. Review the whole repo and fix: sentences over 12 words, hard-coded strings, any data without a LIVE/ARCHIVE label, missing credits, touch bugs, dead ends where a player gets stuck, actions with no cost, anything that shows death, console errors, memory growth over 3 shifts. Run Playwright on three phone sizes and two desktop sizes. Produce docs/QA-report.md.

## M12. Submission pack
> Do milestone M12. Write README.md (what it is, how to play, how it uses NASA data, honest limits), docs/DATASETS.md (each dataset: what, why, how used, LIVE/ARCHIVE, link, one screenshot), and docs/DEMO-SCRIPT.md (a 2-minute video script with timings). Auto-capture screenshots of every key screen into docs/screens. Check every number in the research sheets against the in-game text and list mismatches.

## M13. (Optional) Art upgrade with a free CC0 pack
> Do milestone M13. Only if I give you a folder of a CC0 tileset. Write a loader that swaps procedural textures for the pack's tiles via a mapping file, falling back to the procedural art for anything missing. Update Credits. Do not mix more than one pack.
(Human job: download "Sci-Fi Facility Asset Pack" by murphysdad from OpenGameArt, listed as CC0 with 16x16 tiles, characters and objects. Check the license on its page, unzip into assets-pack/, tell the agent.)

## M14. (Optional) Hindi and Marathi
> Do milestone M14. Add hi.json and mr.json alongside en.json, a language picker, and a Devanagari-capable font via @fontsource (Noto Sans Devanagari). Use the t() function everywhere. Machine-translate, then list every string for a native speaker to review in docs/translation-review.md. Re-run the layout tests for longer strings.

---
## FIX PROMPT (reuse any time)
> Problem: [one sentence: what you did, what you expected, what happened]. Paste any error text exactly. Device/browser: [x]. Find the root cause, fix it with the smallest change, add a test that would have caught it, run `npm run check`, and report in under 10 lines.

## BALANCE PROMPT (after playtests)
> A child played and found it [too hard / too easy / confusing at hour X]. Adjust only src/content/balance.json and the mission JSON, re-run the balance bots, and tell me each number you changed and why.

## REVIEW PROMPT (before submitting)
> Review the whole repo as a strict judge for a NASA kids' education challenge. List the 10 weakest points in priority order, each with the smallest fix. Do not change code yet.
