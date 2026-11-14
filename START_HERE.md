# Outpost Zero: START HERE

You do about 45 minutes of setup once. After that, your job is: paste a prompt, wait, press play, tell the agent what felt wrong.

## Part 1. Install (20 min, one time)
1. **Node.js LTS**: nodejs.org. Check: open a terminal, type `node -v` and `npm -v`. Both must print numbers.
2. **Git**: git-scm.com. Check: `git --version`.
3. **VS Code**: code.visualstudio.com. Install the **GitHub Copilot** extension.
4. **Antigravity**: install it and sign in. Open it on the same project folder later.

## Part 2. Accounts (10 min, one time)
1. **GitHub** account.
2. **Cloudflare** account (free). You will use "Pages" to host the game.
3. **NASA API key**: api.nasa.gov, fill the form, key arrives by email. Keep it private.

## Part 3. Create the project (10 min)
1. On github.com: New repository, name `outpost-zero`, Public, tick "Add a README".
2. In a terminal:
   ```
   git clone PASTE_YOUR_REPO_LINK
   ```
3. Unzip this kit and copy **everything inside the kit folder** into the `outpost-zero` folder (include the hidden `.github` folder).
4. In the terminal:
   ```
   cd outpost-zero
   git add .
   git commit -m "Add kit"
   git push
   code .
   ```

## Part 4. Connect hosting (5 min, one time)
Cloudflare dashboard > Workers & Pages > Create > Pages > Connect to Git > pick `outpost-zero`.
- Build command: `npm run build`
- Output directory: `dist`
- Settings > Variables and Secrets: add `NASA_API_KEY` (type: Secret) with your key.
(Do this after milestone M0 exists, so the first build succeeds.)

## Part 5. The loop (repeat for every milestone in docs/MILESTONES.md)
1. Open a **new chat** in Antigravity (big milestones) or Copilot agent mode (small fixes).
2. Paste the milestone prompt exactly.
3. Wait. The agent writes code, writes tests, runs them and fixes itself.
4. When it says done, run `npm run check` yourself. It must end with ALL GREEN.
5. Run `npm run dev`, open the link, play for 2 minutes.
6. Save your progress: `git add . && git commit -m "M3 done" && git push`
7. If something feels wrong, paste the "Fix prompt" at the bottom of MILESTONES.md with one sentence of what you saw.

## Part 6. Jobs only a human can do
| Job | When | Time |
|---|---|---|
| Watch one kid (age 7-12) play, no hints, take notes | After M5 and after M9 | 30 min each |
| Screen-record a 2-minute demo on your phone | After M9 | 20 min |
| Check the three research files against the links (quick skim) | Before M5 | 30 min |
| Fill in the Space Apps submission form | Last | 30 min |

## If you get stuck
Paste the exact error text into the agent with: "Find the root cause, fix it with the smallest change, run npm run check." Do not edit code by hand.
