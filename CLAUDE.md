# ร้านยาทันใจ (Rx, Please!): notes for Claude

## Who and how
- The owner is a Thai pharmacist building this game. **Always reply in Thai**, concise and friendly.
- Live game: https://nattasitrx.github.io/rx-please/ (GitHub Pages, branch `main`, root). Pushing to `main` updates the site in 1–2 minutes.
- **Only update GitHub.** Do not publish Claude artifacts for this game anymore; the old artifact link is intentionally frozen.
- **Do not do mobile / App Store work** (Capacitor, store builds, font bundling for apps) until the owner says they are ready.
- Content about medicines must stay accurate; ask the owner to double-check new drug facts.

## What the game is
A cute, cooking-game-style pharmacy game in one self-contained file: `index.html` (HTML + CSS + JS, Google Fonts Mali / Sarabun / Pridi / Chakra Petch).
Customers walk in and order medicines; the player prepares them in three zones and taps to hand them over.
- **Tablets:** jar → counting tray → (amber bag for vitamin C, or mortar to crush for patients who can't swallow).
- **Liquids:** dry syrup → water → shake (sediment returns if left) → label; cough syrup and oral syringe are instant.
- **Cold chain:** fridge → ice-pack box with a cold gauge that spoils.

Levels 1–30 exist in this version (`RZ_LAST=30`). After level 30 a "coming soon / ติดตามเวอร์ชันใหม่" card shows. The old hospital OPD pharmacist mode was removed; its last version is kept on branch `archive/opd-mode`.

## Code map (all in index.html)
- Prefix `rz` / `RZ` for everything. Main container `#rz` (`rzEl`) inside `#app` (fixed 1120×630 landscape or 480×900 portrait canvas scaled with `--s`).
- Save: `localStorage['rxplease-rush-v1']` → object `RD` (stars[], best[], wallet, gem, bst{}, up{}, dup{}, gpp{}, cx{}, cxr{}, q{}, w{}, ms{}, life{}, gift{}, bossG{}, seen{}, tutOff, music, hiStart). Always save with `rzSave()` (it also syncs map header pills).
- Debug hook: `window.__rz()` returns the running level state `RZ`.
- Level generator `rzLevel(L, seed)`; unlock schedule `RZ_UNL` (new drugs at L1–8, 14 insulin, 17 crushing, 20 cold eye drops, 23 VIP, 26 enteric-coated aspirin trap, 28 four-item orders).
- Systems: shop upgrades `RZ_UP`; drug cost tiers `RZ_DUPN` (buying smarter lowers cost → more profit, not "services"); my-shop extras `RZ_GPP` (the shop already passes GPP from day one; these are beyond-GPP services); codex `RZ_TOPIC` + per-drug facts, +15/+20 coins for reading each card; daily/weekly quests `RZ_QP` and missions `RZ_MS` with claim-all.
- Gems & boosters: `RZ_BST` (calm, ice, shaker, grinder, aide = pharmacy assistant prepares, ptph = part-time pharmacist serves one customer every 15 s, feast), `RZ_MORE` rescue (+3 customers). Gems only from map gift boxes (after levels 4 and 8 of each chapter), first 3★ on boss levels, weekly quest bonus. Never sold for coins. Free trial levels `RZ_TRIAL`.
- Chapter star gates `rzGateNeed(L)` (about 2.15★ per level needed).
- Support-the-developer card `rzDonate()` (PromptPay QR embedded, `RZ_PPID` e-Wallet ID copy button). Feedback form link goes in `RZ_FORM` (empty = button hidden).
- Art is inline SVG drawn in code (`rzArt`, `czPerson`, `rzScene` shop backdrop, `rzOutdoor` map). Glossy "mobile game" UI skin lives in the CSS blocks near `/* portrait */`.

## Testing
- Syntax: extract the `<script>` and run `node --check`.
- Serve: `python3 -m http.server` in the repo, open with Playwright (Chromium is preinstalled in Claude sessions). Test both 1120×630 and 480×900 viewports and watch `pageerror`.
- A simple bot can play levels by reading `window.__rz()` and clicking `[data-r="src"]`, `[data-r="slot"]` buttons.

## Open items / ideas
- Waiting for the owner's Google Form link → set `RZ_FORM`.
- Analytics: PostHog considered (needs the project `phc_` key); Google Sheets via Apps Script was the alternative.
- Later: app store version (Capacitor, in-app purchases via RevenueCat, login + cloud save, bundle fonts), CrazyGames / Poki with an English option, new chapters after level 30 (e.g. Valentine's night pharmacy: contraception; flu-season pharmacy: test kits, masks).
