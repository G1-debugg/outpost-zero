# Agent rules for "Outpost Zero"

You are building a browser game for kids aged 7-12 that teaches real space-outpost trade-offs by replaying real missions. The human is a complete beginner. Work autonomously. Do not ask questions unless you are truly blocked.

## Read first, every time
1. docs/SPEC.md (the game design; do not change it without being told)
2. docs/MILESTONES.md (find the milestone you were given)
3. src/content/research/*.md (the ONLY allowed source of real-world facts)

## Stack (pinned)
- Phaser 4 (stable line, 4.2.x or newer 4.x; pin the exact version in package.json), TypeScript strict, Vite, Vitest, Playwright, vite-plugin-pwa.
- Phaser 4 changed rendering internals. Do not write Phaser 3 patterns from memory. At M0, copy the `skills/` folder from the phaserjs/phaser GitHub repo into docs/phaser-skills/ and read the relevant files before using any Phaser API. If Phaser 4 blocks you for more than 2 attempts on one problem, tell the human and propose pinning Phaser 3.90.x instead.
- Hosting: Cloudflare Pages. Server code goes in /functions (Pages Functions). Secrets only via environment variables.

## Hard rules
- Look: GBA-era top-down pixel art. Internal resolution 240x160, 16x16 tiles, integer scaling. Original art, names and music only. Never copy any Pokemon asset, name or sound.
- NO downloaded art or audio by default. All sprites are defined in code as palette-indexed pixel strings (src/art/) using docs palette `src/art/palette.json`, and rasterised to textures at boot. All sound is synthesised with WebAudio in src/audio/. Fonts come from npm (@fontsource/pixelify-sans), never a CDN.
- Maps are ASCII text files in src/content/maps/*.txt with a legend in src/content/maps/legend.json. A Vitest test must validate every map (all rooms reachable, all required objects present). Do not use Tiled.
- Simulation code (src/sim) has no Phaser imports and has unit tests.
- All balance numbers live in src/content/balance.json. All text lives in src/content/text/en.json and is read through t("key"). Never hard-code player text.
- Kid text: max 12 words per sentence, about age 9. Jargon uses {{glossary}} markers. `npm run lint:text` must enforce this.
- Exactly three resources: Power, Air, Shield. Every action has a cost and a benefit. No free options. A test must check that no action in any mission JSON has zero cost.
- Failure is never "game over". Use "Mission paused" or "Crew evacuated", then a Debrief and Retry. Never show death.
- Real data is labelled LIVE or ARCHIVE (with date). Never present cached or bundled data as live. Every screen has a Data Sources button.
- Never invent science facts. Only use src/content/research/*.md. If a fact is missing, write TODO(fact) in the content and list it in your report.
- No accounts, no analytics, no tracking, no personal data.
- Keep files small and named clearly. One feature per commit-sized change.

## Definition of done for every milestone
1. `npm run check` passes (typecheck, lint, lint:text, unit tests, build, Playwright smoke test on desktop and a 390x844 phone viewport).
2. You have run the game yourself in a headless browser and looked at a screenshot of the result. Save screenshots to /docs/screens/.
3. Your final message states: what you built, the exact commands to run, what the human should see, and any TODO(fact) or known problems. Keep it under 15 lines.
