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
- Save: `localStorage['rxplease-rush-v1']` → object `RD` (stars[], best[], wallet, gem, bst{}, up{}, dup{}, gpp{}, cx{}, cxr{}, q{}, w{}, ms{}, life{}, gift{}, bossG{}, seen{}, tutOff, music, fs, hiStart). Always save with `rzSave()` (it also syncs map header pills).
- Debug hook: `window.__rz()` returns the running level state `RZ`.
- Level generator `rzLevel(L, seed)`; unlock schedule `RZ_UNL` (new drugs at L1–8, 14 insulin, 17 crushing, 20 cold eye drops, 23 VIP, 26 enteric-coated aspirin trap, 28 four-item orders).
- Systems: shop upgrades `RZ_UP`; drug cost tiers `RZ_DUPN` (buying smarter lowers cost → more profit, not "services"); my-shop extras `RZ_GPP` (the shop already passes GPP from day one; these are beyond-GPP services); codex `RZ_TOPIC` + per-drug facts, +15/+20 coins for reading each card; daily/weekly quests `RZ_QP` and missions `RZ_MS` with claim-all.
- Gems & boosters: `RZ_BST` (calm, ice, shaker, grinder, aide = pharmacy assistant prepares, ptph = part-time pharmacist serves one customer every 15 s, feast), `RZ_MORE` rescue (+3 customers). Gems only from map gift boxes (after levels 4 and 8 of each chapter), first 3★ on boss levels, weekly quest bonus. Never sold for coins. Free trial levels `RZ_TRIAL`.
- Chapter star gates `rzGateNeed(L)` (about 2.15★ per level needed).
- Fullscreen: the splash tap calls `rzFsOn()` on touch devices unless `RD.fs===0` (settings toggle `data-r="fs"`). iPhone Safari has no page fullscreen, so `rzA2hs()` shows an add-to-home-screen guide once (`RD.seen.a2hs`) and in settings. Home-screen app: `manifest.webmanifest` + `icons/` + apple meta tags; `layout()` keeps the canvas inside the safe area (notch). No service worker on purpose, so pushes reach players right away. Guide `rzA2hs()` is an illustrated stepper (`rzPh(scene)` draws SVG phone mock-ups) chosen by `rzEnv()`: Safari 26 (⋯ › Share), older Safari, Chrome iOS, in-app browsers (LINE, Facebook, Messenger, IG, TikTok → open in browser first, shown once per session). LINE is redirected to the external browser at load with `?openExternalBrowser=1` (flag kept in the URL as a loop guard, stripped outside LINE). iOS home-screen apps have separate storage from Safari, so `rzXfer()` moves a save as an `RXP1.<base64 JSON>` code (settings › ย้ายเซฟ; offered on the first home-screen launch via `RD.seen.sa`).
- Analytics: PostHog, wired but off until `RZ_PH_KEY` (phc_…) is set; `RZ_PH_HOST` us/eu. Loader + `rzAnInit()` after `rzSave`; `rzA(event, props)` adds hi_level / levels_cleared / stars_total / orient. Anonymous (localStorage id, no cookies, no autocapture, no recordings); settings toggle `data-r="an"` → `RD.an=0` opts out. Events: $pageview, game_enter, level_start, level_end {result pass/fail/quit/restart/closed, stars, time_s, served, missed, walkout_items{drug:n}, spoiled, trashed, trap_ok/crushed, boosters}, codex_read, shop_buy, feedback_click, donate_open, a2hs_step, home_screen_first_launch, fullscreen_toggle, save_code_copied, save_imported, chapter_gate_blocked, reached_end_of_content. Super props: game_v (`RZ_VER`, bump on releases), display, inapp, from_line_redirect. Testing in headless Playwright needs `opt_out_useragent_filter:true` in a test copy (PostHog drops headless traffic).
- Support-the-developer card `rzDonate()` (PromptPay QR embedded, `RZ_PPID` e-Wallet ID copy button). Feedback form link goes in `RZ_FORM` (empty = button hidden).
- Art is inline SVG drawn in code (`rzArt`, `czPerson`, `rzScene` shop backdrop, `rzOutdoor` map). Glossy "mobile game" UI skin lives in the CSS blocks near `/* portrait */`.

## Testing
- Syntax: extract the `<script>` and run `node --check`.
- Serve: `python3 -m http.server` in the repo, open with Playwright (Chromium is preinstalled in Claude sessions). Test both 1120×630 and 480×900 viewports and watch `pageerror`.
- A simple bot can play levels by reading `window.__rz()` and clicking `[data-r="src"]`, `[data-r="slot"]` buttons.

## Open items / ideas
- Feedback form "📋 ใบสั่งยาจากผู้เล่น" is live (owner's Google account; created by Apps Script) and linked in `RZ_FORM`.
- Later: app store version (Capacitor, in-app purchases via RevenueCat, login + cloud save, bundle fonts), CrazyGames / Poki with an English option, new chapters after level 30 (e.g. Valentine's night pharmacy: contraception; flu-season pharmacy: test kits, masks).
