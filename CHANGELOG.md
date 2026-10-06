# 港鐵到站 — Changelog

> MTR Next Train PWA · https://iceninye.github.io/mtr-app/
> 全部 client-side, 冇 backend, 數據來源: [data.gov.hk MTR Next Train API](https://data.gov.hk/en-data/dataset/mtr-data2-nexttrain-data)

---

## v52.9.0 — 2026-10-06
**Commit:** `9012dcb` · **Change:** accurate station coordinates (nearest-stop fix), MTR failure reason shown, EAL departure times marked

### 🐛 Problems
1. **Wrong nearest LRT stop.** 55 of 68 LRT stops were more than 150 m from where they really are, many by 1–3.8 km (青松 120 was placed next to 屯門碼頭). Standing exactly at a stop, the app picked a *different* stop for **37 of 68** stops, e.g. at 屯門碼頭 it chose 青松, at 青松 it chose 大興 (北), at 麒麟 it chose MTR 兆康.
2. 15 MTR stations were 150–500 m off (坑口 HAH 497 m, 烏溪沙 363 m, 南昌 326 m, 天水圍 314 m …).
3. LRT stop 250 is now **海皇路 / Hoi Wong Road** in MTR's official stop list, not 屯門泳池.
4. A failed MTR request only showed 「無法取得 MTR 實時列車資料」 with no reason, and on a first load / new station left the loading skeleton on screen forever.
5. A few journey-panel strings were Chinese-only in EN mode.

### 🔧 Fix
- LRT and MTR coordinates replaced with surveyed stop locations (hkbus/hk-bus-crawling `stopList`, built from data.gov.hk / MTR open data). Stop names checked against `opendata.mtr.com.hk/data/light_rail_routes_and_stops.csv`. RAC (not in that dataset) unchanged.
- MTR ↔ LRT co-location is now an explicit list (TUM/295, YUL/600, TIS/430, SIH/100) instead of a 150 m radius: with real coordinates TUM↔295 is 155 m and TIS↔430 232 m, but the unrelated TUM↔070 河田 is 252 m.
- `fetchMTRData()` keeps the failure reason (`HTTP 503`, timeout…). It goes into the red banner and the Raw API panel; if nothing real is on screen for that station yet, a "⚠️ 暫時取得唔到港鐵班次 · 原因：…" card replaces the skeleton. A same-station auto-refresh failure keeps the last data under the banner (`state.dataKey`).
- EAL trains with `timeType: "D"` show 「↑ 開出時間」, since their time is a departure, not an arrival.
- **Cloudflare build fixed**: the `Workers Builds: mtr-app` check failed on every commit with `Missing entry-point to Worker script or to assets directory` (reproduced with `wrangler deploy --dry-run`), because the repo had no Wrangler config. Added `wrangler.jsonc` (static assets from the repo root, no Worker script) and `.assetsignore` so only the website files are uploaded (not `.git`, CHANGELOG, Tailwind sources). GitHub Pages is unchanged.
- New i18n keys: `mtrFailTitle`, `checkNetwork`, `noCurrentTrain`, `upstreamNoJourney`, `dataUpdating`, `departsTag`.

### 🧪 Browser-verified (Chromium, mocked API, geolocation set to real stop locations, ±20 m)
| Standing at | v52.8.2 picked | v52.9.0 picks |
|---|---|---|
| LRT 屯門碼頭 001 | 青松 120 | 屯門碼頭 001 |
| LRT 青松 120 | 大興 (北) 212 | 青松 120 |
| LRT 麒麟 110 | MTR 兆康 | 麒麟 110 |
| LRT 海皇路 250 | — | 海皇路 250, chips 豐景園 / 兆麟 |
| MTR 天水圍 exit | — | TIS, chips 🚊 天水圍 430 / 天耀 |
| MTR 屯門 | — | TUM, chips 🚊 屯門 295 / 河田 070 |
| MTR 坑口 | — | HAH |
| 沙田, MTR API HTTP 503 | skeleton + banner without reason | 沙田 card + banner "HTTP 503" |

Live API checked 2026-10-06 23:4x HKT: LRT accepts `station_id=001` and `=1` alike; EAL payload carries `timeType` and `route`.

---

## v52.8.2 — 2026-10-05
**Commit:** `a08ac7a` · **Change:** LRT: no stale MTR card, no red error at night, failure reason shown

### 🐛 Problem (user screenshot, 01:37 HKT)
Auto-located to LRT 天慈 (435), but the page still showed the **previous MTR card (荃灣綫 金鐘 ADM)** plus a red banner 「無法取得 LRT 實時列車資料」. Two causes:
1. A failed LRT request only raised the banner; `#content` was never replaced, so the old MTR station stayed on screen.
2. MTR got the "outside service hours" treatment in v52.8.0, LRT did not (any non-`status: 1` response, an HTTP error or a network error all became the same red error).

> The live LRT API could not be reached from the dev environment, so what it returns at night is **not verified**: `status: 0`, an HTTP error and an empty list are all handled, and the real reason is now displayed.

### 🔧 Fix
- **No stale content**: `showStationLoading()` replaces `#content` with the new station's name + skeleton as soon as the station or system changes (MTR and LRT); `state.renderedKey` tracks what is on screen. Same-station auto-refresh does not flash.
- `fetchLRTData()` now resolves `{ json }` / `{ stale }` / `{ error: reason }`; `status: 0` is a notice (like MTR), a superseded request is never an error.
- `renderLRTUnavailable()` always renders **this** station's card:
  | LRT response | Night (00:30–06:30 HKT) | Day |
  |---|---|---|
  | `status: 1`, no platforms | 🌙 非服務時間 | ℹ️ 暫時冇班次資料 |
  | `status: 0` + message | ⚠️ API message **+** 🌙 hint | ⚠️ API message |
  | HTTP error / network / timeout | 🌙 "API 暫時冇回應 (reason) … 可能已收車", **no red banner** | ⚠️ card with reason **+** red banner with reason |
- The failure reason (e.g. `HTTP 500`) is also in the Raw API metadata panel, so a real daytime problem can be diagnosed.
- New i18n keys: `lrtNoResponseNightBody`, `lrtFailTitle`, `lrtFailBody`, `loadingStation` (zh + en).

### 🧪 Browser-verified (Chromium, mocked API, clock fixed to HKT)
Flow: MTR ADM card on screen → switch to LRT 天慈.
| Case | Result |
|---|---|
| Night, HTTP 500 | 🌙 card + reason, **no banner**, no 金鐘/荃灣綫 left |
| Night, `status: 0` "No data" | ⚠️ message + 🌙 hint |
| Night, empty platforms | 🌙 card |
| Night, network failure | 🌙 card + `Failed to fetch` |
| **Day**, HTTP 500 | ⚠️ card + red banner with `HTTP 500` |
| Day, `status: 0` | ⚠️ message |
| Day, normal data | route list as before |
| Auto-locate at 天慈 ±18 m, 01:37, HTTP 500 | LRT mode, notice + chips, 🌙 card for 天慈, banner hidden |
| Right after tapping LRT (slow response) | "屯門碼頭 載入緊…" skeleton, not the MTR card |

Regression: journey, 2 MTR calls / 25 s, LRT ↔ MTR, v52.7.5 fixes, v52.7.6 chip rules, v52.8.0 no-trains states — unchanged.

---

## v52.8.1 — 2026-10-05
**Commit:** `6dae0c0` · **Change:** selected train card readable in both themes

### 🐛 Problem
The selected card used solid `bg-cyan-400` while its text kept the normal light colours, so in dark theme it was light-on-light. Measured contrast: time 1.6, destination 1.5, date 1.4 (WCAG AA needs 4.5). Light theme: 月台 / date 2.6. After a tap, a sticky `hover:` background could also hide the selection.

### 🔧 Fix
- New `.train-card.is-selected` (custom CSS, no Tailwind rebuild): cyan **tint** + 2 px cyan border
  - dark: `rgba(8,145,178,.28)` + `#22d3ee` border; 月台 label lifted to `#94a3b8`
  - light: `#ecfeff` + `#0891b2` border
- `trainCardStateClass()` is the single source for selected / normal / invalid classes, used by both render and tap; the selected card carries no `hover:` class

### 🧪 Contrast (measured from rendered pixels)
| Text | Dark before → after | Light before → after |
|---|---|---|
| Time (6 分鐘) | 1.6 → **11.3** | 9.9 → **17.2** |
| Destination | 1.5 → **10.1** | 8.1 → **14.1** |
| 往 / code / date | 1.4 → **4.8** | 2.6 → **4.6** |
| 月台 | 2.6 → **4.8** | 2.6 → **4.6** |

Tap #2 → #1 swaps the state classes correctly in both themes; regression (journey base, 2 MTR calls / 25 s, selection across refresh) unchanged.

---

## v52.8.0 — 2026-10-05
**Commit:** `e62f535` · **Change:** clear "no trains" state, PWA icons, iOS safe area / theme colour, best-train default

### 🌙 No-trains state (item 2)
Verified on a phone at **2026-10-05 01:11 HKT** (SIH, TML): after the last train the API returns `status: 1`, `message: "successful"` with **`UP: []`, `DOWN: []`** — the API is fine, there are just no trains. The old UI showed "往 終點站 B (DOWN, 0 trains)", "現無列車服務" and "目前車站暫無班次資料" ×2.

Now one card replaces the direction blocks and journey panels when a station has no trains:
| Response | 00:30–06:30 HKT | Other times |
|---|---|---|
| `UP: []`, `DOWN: []` (real night shape) | 🌙 非服務時間 — API OK ({time}), last train has probably left | ℹ️ 暫時冇班次資料 — API OK, maybe suspended / data missing |
| `data: {}` or key without UP/DOWN | 🌙 非服務時間 | ⚠️ 數據暫時無法取得 (v49.1 upstream case, now i18n'd) |
| `status: 0` + `message` (+ `url`) | ⚠️ 港鐵 API 訊息 + message + 詳情 link | same |

- `status: 0` notices are no longer turned into a generic fetch error; the link is shown only for `https://` URLs.
- One direction empty: title uses the topology terminus (e.g. 往 東涌) instead of "終點站 A/B"; at a station that *is* that direction's terminus (e.g. UP at TUC) the empty block is hidden.
- Journey panel is skipped for a direction without trains; a Train Selector tap re-renders only its own direction.
- LRT: empty platform list at night adds "非服務時間：尾班車可能已經開出。"
- Raw debug panel now distinguishes `(no payload for KEY)`, `"(missing)"` and `[]` (all three used to show `UP: [], DOWN: []`).

### 📱 PWA icons (item 1)
- `assets/icons/`: `icon.svg` (source + favicon), `icon-192.png`, `icon-512.png` (`any`), `icon-maskable-512.png` (`maskable`, glyph inside the 40 % safe zone), `apple-touch-icon.png` (180, opaque)
- Generic train-front design (navy gradient, white body, cyan stripe, amber lights) — **not** the MTR logo (trademark)
- `manifest.webmanifest`: `icons`, `id`, `scope`; `<link rel="icon">` + `<link rel="apple-touch-icon">`

### 🎨 Theme colour + iOS safe area (item 3)
- `theme_color` / `background_color` / dark `theme-color`: `#000000` → `#0f172a` (the app's dark background); the pre-paint script now sets `theme-color` too (the meta tag moved above it)
- `body` padding `env(safe-area-inset-*)` + a fixed `#0f172a` strip behind the status bar: with `black-translucent` the status bar text is white, which was unreadable over the light theme. `env()` is 0 in a normal browser tab, so layout there is unchanged (verified: header/section/footer positions identical)
- ⚠️ Needs a real iPhone (home-screen app) to confirm — headless Chromium cannot emulate the iOS status bar

### 🚆 Best train by default (item 4)
- With no explicit pick, the base train is `pickBestTrain()` (first `valid !== 'N'` with a time) instead of always #1; highlight and `[base #n]` agree
- `sameStation` `-1` fallback kept as is (reviewed: explicit and tested)

### ✅ Item 5 closed
`runLang()` during an in-flight fetch: 6 runs (300 ms / 1.5 s / 4 s delay × MTR/LRT) — final render in the new language, no error banner, superseded response dropped by `requestSeq`.

### 🧪 Browser-verified
| Check | Result |
|---|---|
| Night `UP:[] DOWN:[]` (ZH / EN) | 🌙 card, no direction blocks / journey |
| Day empty arrays | ℹ️ card |
| Day `data: {}` / key without UP/DOWN | ⚠️ upstream card; raw debug shows which |
| Night `data: {}` | 🌙 card |
| `status: 0` + url | API notice + 詳情 link |
| TUC, UP empty | UP block hidden, DOWN + journey shown |
| TSY 00:50, UP empty | 往 東涌 (UP, 0 trains), no UP journey |
| #1 `valid: 'N'` | #2 highlighted, journey `[base #2]` |
| Manifest (Chromium CDP) | 0 manifest errors; icons 192/any, 512/any, 512/maskable all 200 `image/png`; only installability note is `in-incognito` (test browser) |
| `theme-color` dark / light | `#0f172a` / `#f1f5f9`, one meta tag |

Regression: journey, 2 MTR calls / 25 s, LRT ↔ MTR, debug panel, v52.7.5 fixes, v52.7.6 chip rules, train selection across refresh — all unchanged.

---

## v52.7.6 — 2026-10-04
**Commit:** `41d1b84` · **Change:** nearby-station chips — 1500 m "always offer" rule; the selected station is never a chip

### 🎯 Rule
| Nearest station | Auto-pick | Chips |
|---|---|---|
| ≤ 1500 m (`GEO_ALWAYS_OFFER_M`) | nearest, regardless of accuracy | **always** — even at ±10 m / ±100 m |
| 1500 – 2500 m | accuracy gate as before (gap to 2nd > 2 × accuracy) | as before (uncertain, ≤ 50 m, or co-located) |
| > 2500 m | no (unchanged) | — |

- The **selected station is not shown as a chip** — it is already shown and the notice names it. After tapping a chip, the previous station comes back as a chip.
- Re-locate with an uncertain fix keeps the current station; that station is not a chip either.
- The 1000 m accuracy limit and 2.5 km distance limit are unchanged.

### 🧪 Browser-verified (Chromium, scripted geolocation)
| Case | Result |
|---|---|
| ADM ±10 m | ADM picked; chips 中環 780 m / 灣仔 830 m (no 金鐘) |
| ADM ±100 m (previously no chips) | ADM picked; same 2 chips |
| 82 m from ADM, ±600 m (previously "uncertain") | ADM picked, normal notice; 2 chips |
| 2046 m from LMC, big gap, ±100 m | LMC picked by the accuracy gate; no chips |
| 1835 m from LRT 140, gap 147 m, ±100 m | "uncertain" notice, 140 shown; chips 150 / 130 |
| at TKO, re-locate with the case above | TKO kept; chips 140 / 150 / 130 (no TKO) |
| TUM ±15 m | MTR TUM picked; chips LRT 295 / 230 |
| ADM ±10 m → tap 中環 | CEN selected; chips 金鐘 / 灣仔 |
| ADM ±10 m → switch language | chips Central / Wan Chai (still no ADM) |

Regression: journey panel, 2 MTR calls / 25 s, LRT ↔ MTR switch, debug panel, and the v52.7.5 fixes all unchanged.

---

## v52.7.5 — 2026-10-04
**Commit:** `faa50e3`, `79ef180` · **Change:** code-review fixes for v52.4–v52.7.4 (8 bugs, all reproduced in a browser first; + interchange label wording) + CHANGELOG corrections

### 🐛 Fixed
| # | Bug | Fix |
|---|---|---|
| 1 | **Whole app blank when site data is blocked** — `typeof localStorage` / `getItem('mtr-lang')` throws `SecurityError` at top level, so the script stopped | read wrapped in `try/catch`; only `'en'` / `'zh'` accepted |
| 2 | **Language switch drew ADM's 4 line buttons** at any station (and in LRT mode) — `runLang()` read `state.currentStation`, which does not exist | removed the extra `renderSubLinePicker()`; `selectStation()` already renders/hides it for the current station |
| 3 | **大埔墟 shown as "University"** — `STATION_NAMES_EN.TAP` (UNI is University) | `TAP: 'Tai Po Market'` |
| 4 | **Pause button label wrong** — hardcoded Chinese in EN mode, and a language switch reset it to "Pause" while still paused (`data-i18n="pause"`) | `updateAutoToggleLabel()` from `state.autoRefresh` via `t('pause')` / `t('resume')`; `data-i18n` removed from the label |
| 5 | **EN mode: terminus badges blue** — colour came from `label.includes('終點')` | targets carry `isTerminus`; colour uses the flag |
| 6 | **Auto-refresh every 12 s undid the user** — picked train reset to #1 (since v49) and "show more" collapsed | same station: picked train re-found by **dest + ETA (±2 min)**, so it survives the list shifting when a train departs; expanded state kept per direction; both reset on station change. The picked train always stays visible |
| 7 | `<html lang>` stayed `zh-Hant` in EN mode (screen readers used a Chinese voice) | `applyLangChrome()` sets `lang`, title, H1 and toggle label (shared by load + `runLang()`) |
| 8 | Small: duplicate `platformNoTrips` key in both dicts; EN default stop "Tuen Mun Pier" vs station name "Tuen Mun Ferry Pier"; unused `dns-prefetch` for `cdn.jsdelivr.net` | removed / aligned / removed |
| 9 | Interchange line buttons were titled 「支線：」/ "Branch:", but they are different lines, not branches | 「綫路：」/ "Lines:" |

### 📝 CHANGELOG corrections
- Entries are now newest-first (v52.7.x had been placed below v50.x).
- v52.4 – v52.6 dates: 2026-10-04 (commits are 2026-10-04 +0800), not 10-03.
- v52.3, v52.2, v52.1, v52, v51.1, v51 and the v40–v50 summary rewritten from the actual commits (see each entry).
- v52.4 privacy note: removed the unverified PDPO "informed consent" claim; the footer line is a description, not a consent mechanism.

### 🧪 Browser-verified (Chromium, mocked API + scripted geolocation)
| Check | Before | After |
|---|---|---|
| localStorage throws | blank page, `SecurityError` | 將軍澳 (TKO) loads, no errors |
| TKO → switch language | ADM line buttons shown | picker hidden |
| LRT mode → switch language | ADM line buttons shown | picker hidden |
| ADM → switch language | — | its 4 lines, correct |
| EN pause label | `▶ 恢復自動更新` | `▶ Resume Auto-Refresh` |
| Paused → switch to ZH | `⏸ 暫停自動更新` (wrong) | `▶ 恢復自動更新` |
| EN terminus badges | all BLUE | Terminus RED, Interchange BLUE |
| TAP in EN | University (TAP) | Tai Po Market (TAP) |
| `<html lang>` in EN | `zh-Hant` | `en` |
| Pick #3, expand, auto-refresh | back to #1, collapsed | still #3, still expanded |
| First train departs | — | same train (08:03 → LHP) followed from #3 to #2 |
| Change station | — | selection + expansion reset |

Regression: journey panel, 2 MTR calls / 25 s, LRT ↔ MTR switch, debug panel, and geolocation scenarios A–G all unchanged.

---

## v52.7.4 — 2026-10-04
**Commit:** `a0f1bea` · **Change:** remove the Auto mode button — auto is the initial state only

### What changed
Ice reviewed v52.7.3 and asked for the `🔄 自動` button **not to be displayed**. The chosen option was **完全移除 auto 功能；auto 只係首次載入狀態（無法返去）**.

Removed:
- the `<button id="mode-auto" data-mode="auto" data-i18n="autoMode">🔄 自動</button>` element
- its click listener (`() => setMode('auto')`)
- the dictionary keys `autoMode` + `modeAutoAria` (zh + en) → **95 → 93 keys, still full parity**
- the `if (mode === 'auto')` branch inside `setMode()` (unreachable again) and the auto guard on `userSelectedMode`

`setMode()` is now a strict two-state function:
```js
function setMode(mode) {
  // v52.7.4 — auto is only the initial state (state.userSelectedMode === null on load).
  // There is no UI path back to it, so only 'MTR' / 'LRT' reach here: always sticky.
  state.userSelectedMode = mode;
  state.mode = mode;
  …
}
```

### What deliberately did NOT change
`userSelectedMode` is **kept** — it is not just auto-button plumbing. It is what makes first-load auto-detection work: on load `state.userSelectedMode === null`, so `syncMode()` lets the *geolocation-detected* system win. Tapping 重鐵/輕鐵 sets it, making the choice sticky. Removing it would let every later fix silently override the user's manual choice.

Also kept: `state.mode = 'auto'` as the documented initial value, and `tryGeolocate(true)` on `DOMContentLoaded`. **Auto behaviour on first load is unchanged** — only the way back from a sticky override is gone (a reload returns to auto).

### Consequence accepted
After tapping 重鐵 or 輕鐵 there is **no UI path back to auto** other than reloading the page. This is intentional per Ice's decision, and it supersedes item ② of v52.7.3.

### 🧪 Browser-verified
| Check | Result |
|---|---|
| Mode buttons in DOM | 2 only — `重鐵 / MTR`, `輕鐵 / LRT` ✅ |
| `#mode-auto` still exists? | `false` ✅ |
| Any auto-mode label rendered (zh + en) | 0 ✅ |
| Remaining `🔄` / `自動` / `Auto` hits | all pre-existing & unrelated: `🔄 刷新/Refresh` button, `🔄 Recheck` debug button, `下次自動更新/Next auto-refresh:` label, `⏸ 暫停自動更新/Pause Auto-Refresh` toggle ✅ |
| Initial load state | `userSelectedMode = null`, `state.mode = 'auto'` (auto intact) ✅ |
| Click 輕鐵 → sticky | `LRT`, `aria-pressed` = MTR:false / LRT:true ✅ |
| Click 重鐵 → sticky | `MTR`, `aria-pressed` = MTR:true / LRT:false ✅ |
| v52.7.3 chips still work | EN `Admiralty · 0 m`(active)/`Central · 705 m`/`Hong Kong · 899 m` ✅ |
| Chips names follow language | ZH → `金鐘/中環/香港` ✅ |
| Removed keys fully purged | `autoMode` absent from both dicts ✅ |
| Dictionary parity | **93 zh / 93 en** ✅ |
| Syntax / class audit | OK · no new utility classes (no CSS rebuild) ✅ |

---

## v52.7.3 — 2026-10-04
**Commit:** `d99414b` · **Feature/Fix:** reachable Auto mode + always offer the next 2 nearest stations

### ① `'附近車站：'` hardcoded — fixed
```js
// before
box.innerHTML = '<span class="text-xs text-slate-400 self-center">附近車站：</span>' + …
// after
box.innerHTML = `<span …>${escapeHtml(t('nearbyStations'))}</span>` + …
```
Same bug class as the v52.7.2 error banner: `t('nearbyStations')` already existed in the dictionary but was never used, so the chips label stayed Chinese in EN mode.

**Why the v52.7.2 audit missed it:** `renderGeoChoices()` only runs on a *successful* fix that produces candidates. The audit ran with the location permission denied, so that `<span>` never rendered. **Lesson: an empirical scan only covers the code paths the test actually exercises.**

### ② Auto mode was unreachable — fixed
`setMode('auto')` had zero callers (`grep` found only the definition plus the two `MTR`/`LRT` button handlers). Consequence: on load `state.mode = 'auto'` / `userSelectedMode = null`, but **tapping 重鐵 or 輕鐵 made the override sticky forever** — there was no UI way back to auto short of a page reload, and the `if (mode === 'auto') { tryGeolocate(true); return; }` branch was dead code.

Fix: a third mode button.
```html
<button id="mode-auto" type="button" data-mode="auto" data-i18n="autoMode" data-i18n-aria="modeAutoAria">🔄 自動</button>
```
**⚠️ Superseded in v52.7.4** — the `🔄 自動` button was removed at Ice's request; auto mode is now the initial state only. The analysis below is kept as a record of the v52.7.3 state.

`setMode('auto')` needed no logic change — it was already correct (`userSelectedMode = null` → the existing `tryGeolocate(true)` path). Note `state.mode = 'auto'` matches the documented initial state (`mode: 'auto', // 'auto'|'MTR'|'LRT'`), and every `state.mode === 'LRT'` check falls through to the MTR branch safely; the first successful fix self-corrects `state.mode` via `syncMode()`.

### ③ Good fix now always offers the next 2 nearest stations
Previously chips appeared **only** when `!confident` (gap ≤ 2 × accuracy) or when MTR/LRT shared a location. Now, after passing all gates:
```js
const choices = cands.slice(0, 3);
coLocated.forEach(c => { if (!choices.includes(c)) choices.push(c); });
if (accuracy <= GEO_GOOD_ACCURACY_M || coLocated.length) {
  renderGeoChoices(choices, top.code);
} else {
  clearGeoChoices();   // also drops stale chips from an earlier fix
}
```
So a fix at **≤ 50 m** accuracy always shows the nearest station (marked active) **plus the next 2 nearest** for one-tap correction — no need to re-locate.

### ④ Bonus: chips no longer go stale on a language switch
While fixing ① it turned out `runLang()` never re-rendered the chips, so their label and station names stayed in the previous language (the same staleness family as the deferred `geo-notice` issue). Fixed for the chips:
- `state.lastGeoChoices = { cands, activeCode }` stored on render, nulled by `clearGeoChoices()`
- `runLang()` re-invokes `renderGeoChoices()` from the stored state
- new `geoCandidateName(c)` resolves the display name at **render** time (MTR via `stationName()`, LRT via `LRT_STATIONS` + `lrtStationName()`) instead of baking the fix-time language into `c.name`

`geo-notice` itself is **still** not re-translated on a language switch — that remains accepted/deferred (it needs a re-render closure through 14 `setGeoNotice()` call sites plus `defaultNotice` in `handleGeoFix()`).

### 🌐 Dictionary
New keys (zh + en): `autoMode`, `modeAutoAria` → **95 zh / 95 en, full parity.**
No new utility classes → `assets/tailwind.css` rebuild not required (class audit re-run: still 182 styled classes).

### 🧪 Browser-verified (real clicks + synthetic fixes)
| Check | Result |
|---|---|
| 3 mode buttons | `Heavy Rail` / `Light Rail` / `🔄 Auto` ✅ |
| 輕鐵 click → sticky | `userSelectedMode = 'LRT'` ✅ |
| 自動 click → back to auto | `userSelectedMode = null`, `state.mode = 'auto'`, auto button `aria-pressed=true` ✅ |
| Chips at 20 m accuracy | 3 chips — `Admiralty · 0 m` (active) / `Central · 705 m` / `Hong Kong · 899 m` ✅ |
| Chips at exactly 50 m | 3 chips ✅ (`<=`) |
| Chips at 100 m (confident) | cleared → 0 chips, hidden ✅ |
| Chips at 500 m (unconfident) | 3 chips via the pre-existing `!confident` path ✅ |
| Chip click | station → `TWL::CEN`, notice `Heavy Rail station Central (CEN) selected.` ✅ |
| Chip click keeps auto mode | `userSelectedMode` stays `null` ✅ |
| MTR names EN → ZH on lang switch | `Admiralty/Central/Hong Kong` → `金鐘/中環/香港` ✅ |
| LRT names ZH → EN on lang switch | `屯門碼頭 (001)/青松 (120)/美樂 (010)` → `Tuen Mun Ferry Pier/Ching Chung/Melody Garden` ✅ |
| Active chip survives lang switch | index unchanged ✅ |
| Chips stay hidden after `clearGeoChoices()` + lang switch | ✅ |

---

## v52.7.2 — 2026-10-04
**Commit:** `08523b1` · **Fix:** i18n completeness — EN aria-labels + error banner

### 🔍 Audit method
Load the app in EN mode and walk **every** element plus its `aria-label` / `title` / `placeholder` / `alt`, counting CJK characters that should not be there. Static scanning proved too noisy (most CJK hits were legitimate `currentLang === 'en' ? … : …` ternaries), so **the empirical DOM scan is the authoritative check**. It found **28 CJK leaks** in EN mode.

### 🎯 Fixed (28 → 8; remaining 8 all intentional)
| # | Problem |
|---|---|
| 1 | **10 static `aria-label` never translated** — `mode-mtr`, `mode-lrt`, `geo-btn`, `refresh-btn`, `geo-choices`, `auto-toggle`, `dbg-recheck`, `dbg-request`, `dbg-revoke`, `retry-btn` |
| 2 | **Error banner fully Chinese in EN mode** — the `<p>` had no `data-i18n`, and the retry button text + aria were hardcoded, even though `t('loadError')` and `t('retry')` **already existed in the dictionary but were never used** |
| 3 | **Train-card reference aria-label mixed zh+en** — `揀選此卡車 #1 為 To Chai Wan reference train (UP)` |

### 🔧 Mechanism (reusable, matches the existing `data-i18n` pattern)
```html
<button data-i18n-aria="retryAria">…</button>
```
`applyUIText()` now also walks `[data-i18n-aria]` and syncs `aria-label`, so a11y labels follow the UI language on load **and on every `runLang()` switch**. Train-card aria goes through `t('trainCardAria', {seq,title,dir})`.

### 🌐 Dictionary
New keys (zh + en): `modeMtrAria`, `modeLrtAria`, `geoBtnAria`, `refreshBtnAria`, `autoToggleAria`, `retryAria`, `dbgRecheckAria`, `dbgRequestAria`, `dbgRevokeAria`, `trainCardAria`
→ **93 zh / 93 en, full parity.**

Intentionally bilingual (v52.6.7 EN-primary + Chinese-in-parens policy): lang-toggle aria, `EN/中` label, line badges/buttons (`Island Line (港島綫)`), debug-panel iOS note.

**No new utility classes** → `assets/tailwind.css` did not need a rebuild (class audit re-run: still 182 styled classes).

### 🧪 Browser-verified
| Check | Result |
|---|---|
| EN error banner | `⚠️ Error fetching train data` ✅ |
| EN retry button + aria | `Retry` / `Retry fetching train data` ✅ |
| EN 10 aria-labels | all English ✅ |
| ZH error banner + retry | `⚠️ 取得列車資料時發生錯誤` / `重試` / `重試取得列車資料` ✅ |
| Live lang toggle both ways | ✅ |
| Theme preference untouched by lang switch | ✅ (`system`) |
| Lang toggle rebuilds `<select>` | ✅ `堅尼地城（KET · ISL）` → `Kennedy Town (KET · ISL)` |

### ⚠️ Known issue (accepted, NOT fixed)
The **`geo-notice` text is rendered once at event time**, so after a language switch it stays in the old language until the next geolocation event. Fixing it requires threading a re-render closure through **14 `setGeoNotice()` call sites** plus `defaultNotice` through `handleGeoFix()` — that risk in the geolocation path is **not justified** by a cosmetic staleness that self-heals on the next location event. Accepted as a known issue (2026-10-04 mandate).

---

## v52.7.1 — 2026-10-04
**Commit:** `aed8ab4` · **Fix:** rebuild `assets/tailwind.css` — restore missing utility classes

### 🐛 Root cause
The compiled stylesheet was built **once** (v51.1 VUL-09 local build) and never rebuilt, so **every utility class added to `index.html` afterwards was silently absent**. No CSS means no styling and **no console error** — the failure is invisible.

### 🔍 Audit method
Extracted all **190** distinct classes from `index.html` — `class="..."` attributes including those inside JS template literals, plus `classList.add/remove/toggle()` literals — and tested each against the compiled selectors, excluding inline-`<style>` custom classes and Tailwind's `group` marker (which intentionally emits no rule).

### 🎯 Gaps found and fixed
| Missing class | Visible symptom |
|---|---|
| `bg-cyan-400` | Selected train card had **no fill** — transparent background, cyan border only — in **both** themes |
| `focus:ring-cyan-400` | Theme / lang toggle buttons lost their cyan keyboard focus ring |
| `hover:bg-slate-700/70` | Theme / lang toggle buttons had **no hover background** |

### 🔧 Fix
Regenerate from source with the pinned local CLI (v3.4.13):
```bash
npx tailwindcss -i assets/tailwind.src.css -o assets/tailwind.css --minify
```
- `assets/tailwind.css` **16,377 → 17,397 bytes**; styled selectors **178 → 182**
- content globs in `tailwind.config.js` already cover `./index.html`

### 🛡️ Regression safety
- Selector-set diff old vs new: **0 styled selectors lost**
  (`bg-cyan-700` dropped, but has **0** references in `index.html` — stale leftover in the old build)
- 5 spot-checked dark rules **byte-identical** before/after: `.bg-slate-800/60`, `.text-slate-100`, `.glass`, `.bg-cyan-600`, `.text-amber-300`
- 13 selectors added, all newly used

### 🧪 Browser-verified (local server, transitions disabled for reliable measurement)
| Card | Dark | Light |
|---|---|---|
| `#1` selected | `rgb(34,211,238)` bg / `rgb(15,23,42)` fg ✅ | `rgb(34,211,238)` bg / `rgb(15,23,42)` fg ✅ |
| `#2`–`#4` | `rgba(30,41,59,0.6)` / `rgb(226,232,240)` (unchanged) | `rgba(255,255,255,0.78)` / `rgb(15,23,42)` |

Toggle buttons' hover + focus utilities now resolve (6 selectors).

### 📌 Lesson
Any change to utility classes in `index.html` **requires** rebuilding `assets/tailwind.css`, otherwise the class silently does nothing. Worth a pre-deploy check.

---

## v52.7.0 — 2026-10-04
**Commit:** `e014680` · **Feature:** 3-state theme toggle (light / dark / system)

### ✨ Placement
Header right cluster: `Live dot | #theme-toggle | #lang-toggle` (single button, emoji only).

### 🌗 Theme model
| Pref | Icon | Resolved |
|---|---|---|
| `system` (default) | 💻 | follows `prefers-color-scheme` |
| `light` | ☀️ | `data-theme="light"` |
| `dark` | 🌙 | `data-theme="dark"` |

- localStorage key **`mtr-theme`**, values `light` / `dark` / `system`, default `system`
- `<html data-theme="light|dark">` holds the **resolved** theme; `color-scheme` set alongside
- Inline **pre-paint script** in `<head>` resolves the theme before first paint (no flash)
- `meta[name="theme-color"]` updated on switch (`#000000` dark / `#f1f5f9` light)

### 🧩 JS API
```js
getStoredTheme()            // read localStorage, fall back to 'system'
getResolvedTheme(pref)      // system -> matchMedia, else pref
applyTheme(pref)            // persist + set data-theme + color-scheme + button UI
cycleTheme()                // system -> light -> dark -> system
```
`matchMedia('(prefers-color-scheme: dark)')` change listener fires **only when pref === 'system'**.

### ♿ Button a11y
`id="theme-toggle"`, `aria-label` + `title` from `t('themeToggleAria', {mode})`.
Refreshed by `runLang()` so the aria text follows the UI language; theme never resets on language switch.

### 🎨 CSS light overrides (dark styles untouched)
- `body` gradient → light slate `#f1f5f9`; `.glass` → white translucent; `.skeleton`
- slate text `100/200/300/400/500/600` remapped darker for light surfaces
- accent text 200/300 → 700 shades (amber / emerald / cyan / orange / blue / red / green)
- slate + red / amber / blue surfaces and borders → light equivalents
- solid `bg-slate-800 / 700 / 600` + hover / active variants → light surfaces
- colored buttons (`bg-blue-*`, `bg-red-*`, `bg-cyan-600`) forced **white text** in light mode
- `select` / `input` / `option` get white bg + dark text
- removed the vestigial `class="dark"` on `<html>` (no `.dark` rules exist in the compiled CSS)

### 🌐 i18n keys added (zh + en)
`themeLight` / `themeDark` / `themeSystem` / `themeToggleAria` → dict now **83 zh / 83 en**, full parity.

### 🔧 Included fix
Countdown number + unit were split by a `justify-between` row and drifted apart. Wrapped with the label in one span → reads `下次自動更新： 7 sec`.

### 🧪 Verified in a real browser (local http server)
| Check | Result |
|---|---|
| Light mode contrast (< 3.0 ratio scan) | ✅ 0 elements |
| Dark mode contrast | ✅ 0 elements |
| Dark look unchanged (body `rgb(15,23,42)` / text `rgb(226,232,240)`) | ✅ |
| Cycle via real click | ✅ system → light → dark → system, storage follows |
| Persistence across reload (saved `light`) | ✅ `data-theme=light`, icon ☀️ |
| `system` + OS change | ✅ live update; `pref=light` ignores OS change |
| Invalid stored value falls back to `system` | ✅ |
| Lang toggle still rebuilds select + EN-primary | ✅ `堅尼地城（KET · ISL）` → `Kennedy Town (KET · ISL)` |
| Theme survives lang toggle | ✅ pref unchanged |

### ⚠️ Pre-existing issue found (NOT introduced here, out of scope per locked spec)
`bg-cyan-400` is **absent** from the compiled `assets/tailwind.css`, so the selected train card renders a transparent background with a cyan border only — in **both** themes. Reported for a follow-up decision.

---

## v52.6.12 — 2026-10-04
**Commit:** `6f9c4c2`

### 🐛 Hardcoded `精度 ±${acc} m` leaked to EN mode

User report: 「定位至最近重鐵車站 沙田圍（STW），距離 0.03 km（精度 ±49 m）。」 — even after toggling to EN, the **「精度 ±49 m」 suffix stayed in Chinese**.

Root cause: line 2069 hardcoded the accuracy text:
```js
const accText = `精度 ±${Math.round(accuracy)} m`;
```

This `accText` was then interpolated into 4 templates via `{acc}` placeholder (`geoLocated`, `geoUncertainShow`, `geoUncertainKeep`, `geoTooFar`). Those templates have proper EN translations, but `accText` itself bypassed the i18n layer.

### Fix
- Added `geoAccuracyUnit` dict key (zh + en)
- Replaced the hardcoded literal with:
  ```js
  const accText = t('geoAccuracyUnit', { acc: Math.round(accuracy) });
  ```

### New dict keys
| Key | ZH | EN |
|---|---|---|
| `geoAccuracyUnit` | 精度 ±{acc} m | accuracy ±{acc} m |

### Verify
```
ZH geoAccuracyUnit (acc=49): 精度 ±49 m
EN geoAccuracyUnit (acc=49): accuracy ±49 m
```

---

## v52.6.11 — 2026-10-04
**Commit:** `3e286e7`

### 🔧 2 fixes from user mandate

**Issue 1: countdown unit mashed with number**

Before: `下次自動更新： 7s` (number + 's' together, looks like 7s = 7 seconds as one token)

After: `下次自動更新： 7 sec` — split into two spans:
```html
<span id="countdown">7</span><span id="countdown-unit">sec</span>
```
Both update on `runLang()` (lang toggle) and on countdown restart.

### 🧪 Issue 2: UP/DOWN trains all visible (too long)

Before: all 4 train cards rendered at once.

After: show first 2 (`#1` + `#2`), hide `#3`+`#4` behind `▾ Show N more` button:
- Hidden trains get `.hidden .train-collapsed` class
- Button: `<button onclick="this.previousElementSibling.querySelectorAll('.train-collapsed').forEach(el=>el.classList.remove('hidden'));this.classList.add('hidden');">▾ Show N more</button>`
- After click, button hides itself

### 📊 4 new dict keys (zh + en)
- `showMoreTrains`: `顯示其餘 {n} 班` / `Show {n} more`
- `secondsShort`: `sec` / `sec`

### 🧪 Verify (mock browser env)

| Test | Result |
|---|---|
| Render with 4 trains → #1 + #2 visible | ✅ |
| Render with 4 trains → #3 + #4 hidden | ✅ (2× `hidden train-collapsed` match) |
| Show 2 more button text | ✅ `▾ Show 2 more` |
| Countdown number / unit separated | ✅ two `<span>` elements |

### 🐛 Bug fix during edit
- Patch tool fuzzy-matched too much on `renderDirectionBlock` and dropped `isValid`/`isArriving` local vars. Restored manually.

---

## v52.6.10 — 2026-10-04
**Commit:** `6f3c5a4`

### 🔧 Plain modeLabel — strip emoji + parens from geo notice

User report:
- 期望: `已定位至最近**重鐵**車站 錦上路（KSR），距離 0.11 km（精度 ±11 m）。`
- 之前: `已定位至最近🚇 重鐵 (Heavy Rail)車站 錦上路...`（多 emoji + parens）

### Fix
- `I18N_UI.zh.mtr`: `🚇 重鐵 (Heavy Rail)` → `重鐵`
- `I18N_UI.zh.lrt`: `🚊 輕鐵 (Light Rail)` → `輕鐵`
- `I18N_UI.en.mtr`: `🚇 Heavy Rail (重鐵)` → `Heavy Rail`
- `I18N_UI.en.lrt`: `🚊 Light Rail (輕鐵)` → `Light Rail`

`{modeLabel}` placeholder 注入 `geoLocated` / `geoUncertainShow` / `geoSelected` templates。

### 🧪 Verify (mock browser env)

| Call | EN | ZH |
|---|---|---|
| `t('mtr')` | Heavy Rail | 重鐵 |
| `t('lrt')` | Light Rail | 輕鐵 |
| `t('geoLocated', {modeLabel: t('mtr'), name: 'KSR EN', code: 'KSR', dist: '0.11', acc: '±11 m'})` | Located to nearest **Heavy Rail** station **Kam Sheung Road (KSR)**, 0.11 km away (±11 m). | 已定位至最近**重鐵**車站 **錦上路（KSR）**，距離 0.11 km（±11 m）。 |

EN station name confirmed via `STATION_NAMES_EN['KSR'] = 'Kam Sheung Road'` (line 641 in dict).

---

## v52.6.9 — 2026-10-04
**Commit:** `cc11a48` · **Live:** 114,254 bytes

### 🐛 CRITICAL FIX — t() shadow in `renderDirectionBlock` + `renderLastUpdate`

**Bug:** `t is not a function. (In 't('platform')', 't' is an instance of Object)`

User-reported after v52.6.8 deploy: train card section threw `[object Object]` instead of rendering `Platform` label.

### Root cause — two `t` shadow sites

| Function | Shadow | Symptom |
|---|---|---|
| `renderLastUpdate(now)` | `const t = now \|\| new Date()` | t() returned Date object → `[object Object]` (would also affect any t() call here) |
| `renderDirectionBlock(trains,...)` | `trains.map((t, idx) => ...)` | t was a train object; every `t('platform')` returned train object → throw + no Platform label |

Both used `t` as a local var / loop param, shadowing the global `t()` function from `I18N_UI` dict lookup.

### Fix
- `renderLastUpdate`: rename local `t` to `nowRef`
- `renderDirectionBlock`: rename loop var `t` → `tr`; update 11 `t.` references inside callback to `tr.`

### Verify
```
Render contains "Platform": true (EN mode)
Render contains "月台": false
Render contains "[object Object]": false
Render contains "t('platform')": false
```

### Related lessons (re-applied from v52.6.6)
- ❌ NEVER use `const t`, `let t`, or `t` as loop param when global `t()` exists in scope
- ✅ Use `now`, `tr`, `entry`, `item`, etc.

---

## v52.6.8 — 2026-10-04
**Commit:** `4929799` · **Live:** 114,064 bytes

### 🔧 EN-mode leftovers — 4 fixes

**1. `runLang()` dropdown rebuild order**
- Before: `selectStation()` was called first → dropdown rebuilt with stale options (EN mode showed 金鐘)
- After: `manualStationSelect()` runs first → options rebuild in `currentLang` → dropdown shows `Admiralty (ADM · ISL)` in EN mode

**2. Geo notice i18n**
- `'定位資料無效，'` → `t('geoInvalidCoords')`
- `'定位成功但找不到有效車站，'` → `t('geoNoStation')`

**3. MTR direction block `月台` → `t('platform')`**
- Was hardcoded Chinese in `renderDirectionBlock()` label
- Now: `Platform` (EN) / `月台` (ZH)

**4. Cars pluralization**
- Was: `${trainLen} cars` (always plural)
- After: `t('{n} {cars}', {n: trainLen})` — `{cars}` placeholder resolves to `cars` / `carsPlural` based on `n`

### 📊 4 new dict keys
- `geoInvalidCoords`, `geoNoStation` (new geo strings)
- `platformNoTrips` (UI string for empty platform)
- `carsPlural` (plural variant for English)

### 🧪 Verify (mock browser env)

| Call | EN | ZH |
|---|---|---|
| `t('geoInvalidCoords')` | Invalid location data, | 定位資料無效， |
| `t('geoNoStation')` | Location succeeded but no valid station found, | 定位成功但找不到有效車站， |
| `t('platform')` | Platform | 月台 |
| `t('platformNoTrips')` | no upcoming trips | 暫無班次 |
| `t('{n} {cars}', {n:1})` | 1 car | 1 卡 |
| `t('{n} {cars}', {n:3})` | 3 cars | 3 卡 |

Total dict keys: **76 zh + 76 en**

---

## v52.6.7 — 2026-10-04
**Commit:** `79ac9f2` · **Live:** 113,257 bytes

### 🌐 EN-primary labels — Chinese only in parentheses (product rule)

**Rule:** `currentLang === 'en'` → every user-visible string English-first; Chinese only as secondary in parens. `currentLang === 'zh'` → Chinese primary.

### 🛠️ Helpers added / extended
- **`stationNameBilingual(code)`** — EN `Mei Foo (美孚)` / ZH `美孚`
- **`lrtStationName(s, bilingual=true)`** — EN `Lam Tei (藍地)` / ZH `藍地`
- **`lineName(obj|code, bilingual=true)`** — now accepts a **line code string** OR LINE_COLORS object; EN `Tsuen Wan Line (荃灣綫)`
- **`systemLabel(mode)`** — EN `Light Rail (輕鐵)` / `Heavy Rail (重鐵)`; ZH `輕鐵` / `重鐵`
- **`bilingual(en, zh)`** — generic EN(中) pairing helper

### 🔧 Chinese-first sites fixed

| Site | Before (EN mode) | After (EN mode) |
|---|---|---|
| LRT badge + optgroup | 輕鐵 (Light Rail) | **Light Rail (輕鐵)** |
| LRT station option | 藍地（350 · Lam Tei） | **Lam Tei (藍地 · 350)** |
| MTR line optgroup | 屯馬綫（TML · 27 站） | **Tuen Ma Line (屯馬綫) — TML · 27 stations** |
| MTR station option | 美孚（MEF · TWL） | **Mei Foo (MEF · TWL)** (halfwidth parens) |
| Update time | 更新於 HH:MM:SS (HKT) | **Updated HH:MM:SS (HKT)** |
| LRT route card dest | 往 新圍 (San Wai) | **To San Wai (新圍)** |
| LRT route card unit | `1卡` | **`1 cars`** |
| LRT route time | time_ch | **time_en** |
| LRT platform badge | ↓ 到站 / ↑ 開出 | **↓ Arriving / ↑ Departing** |
| Footer source | 資料來源：(literal, key unused) | **data-i18n="dataSource"** wired |
| Mode buttons | Heavy Rail / Light Rail | **Heavy Rail (重鐵) / Light Rail (輕鐵)** |
| Nearby LRT candidate name | name_tc | **EN-first in EN mode** |

### 📊 Stats
- 2 new dict keys (`arrivingShort`, `departingShort`)
- Total dict keys: **72 zh + 72 en**

### 🧪 Verify (mock browser env)

| Call | EN | ZH |
|---|---|---|
| `stationNameBilingual('MEF')` | Mei Foo (美孚) | 美孚 |
| `lrtStationName(LRT_STATIONS['350'])` | Lam Tei (藍地) | 藍地 |
| `lineName('TWL')` | Tsuen Wan Line (荃灣綫) | 荃灣綫 |
| `systemLabel('LRT')` | Light Rail (輕鐵) | 輕鐵 |
| `dirWord()` | To | 往 |
| `t('dataSource')` | Data source: | 資料來源： |

---

## v52.6.6 — 2026-10-04
**Commit:** `783f6b6` · **Live:** 110,743 bytes

### 🌐 EN-first i18n — all user-visible strings respect `currentLang`

### 🛠️ Helper functions added (avoid bypass patterns)
- **`lrtStationName(s)`** — use for LRT station display (was buggy direct `.name_tc`/`.name_en` with **reversed logic** in zh mode)
- **`dirWord()`** — returns `To` (EN) or `往` (ZH) for direction blocks

### 🔧 Hardcoded bypass patterns replaced

| Issue | Location | Fix |
|---|---|---|
| LRT header bypass + reversed logic | `@67286, @67455` | `lrtStationName(station)` + reverse subtitle |
| Sub-line picker (`lcInfo.name_tc` always Chinese) | `@73809` | `lineName(lcInfo)` |
| Service status (服務正常/延誤) | `@1154-1155` | `t('serviceNormal')` / `t('serviceDelay')` |
| 「往」 hardcoded | `@1048, @1087, @1303` | `dirWord()` |
| UP/DOWN (上行/下行) | `@1235, @1246, @1610` | `t('dirUp')` / `t('dirDown')` |
| 終點站/轉車站/主要站 | `@464, @470, @474, @1204-1205, @1314, @1614, @1625` | `t('terminus')` etc. |
| 計算中/失敗 | `@1243, @1246` | `t('computing')` / `t('computeFail')` |
| 現站 API 失敗 | `@1631` | `t('apiFailCurrent')` |
| 同路線站間動態行車時間 / 當前列車到 | `@1639` | `t('dynamicJourney')` / `t('curTrainAt')` |
| 本方向無下游車站 | `@1624` | `t('noDownstream')` |
| 無法取得現有車站 API 資料 | `@1627` | `t('apiFailNoData')` |
| LRT 站 X 不存在 | `@1292, @1693` | `t('lrtStationNotFound')` / `t('lrtStationNotFoundMap')` |
| 無法取得 LRT/MTR 實時列車資料 | `@1697, @1710` | `t('lrtFetchFail')` / `t('mtrFetchFail')` |

### 📊 20 new dict keys (zh + en)
- `serviceDelay` / `serviceNormal`
- `dirUp` / `dirDown`
- `computing` / `computeFail` / `apiFailCurrent` / `dynamicJourney` / `curTrainAt`
- `terminus` / `terminusInterchange` / `interchange` / `majorStop`
- `terminusANoData` / `terminusBNoData` / `reachedTerminus` / `noTargets`
- `noDownstream` / `apiFailNoData`
- `lrtStationNotFound` / `lrtStationNotFoundMap` / `lrtFetchFail` / `mtrFetchFail`

### 📈 Stats
- Total `t()` calls: **65** (was 42, **+23**)
- Total dict keys: **70 zh + 70 en** (was 50, **+20 each**)

### 🧪 Verify (mock browser env)

| Function | EN | ZH |
|---|---|---|
| `t('serviceNormal')` | ✓ Service Normal | ✓ 服務正常 |
| `t('terminus')` | Terminus | 終點站 |
| `t('dirUp')` | UP | 上行 |
| `dirWord()` | To | 往 |
| `lrtStationName({name_en:"Mei Foo", name_tc:"美孚"})` | Mei Foo | 美孚 |
| `lineName({name_en:"Tsuen Wan Line", name_tc:"荃灣綫"})` | Tsuen Wan Line | 荃灣綫 |

---

## v52.6.5 — 2026-10-04
**Commit:** `d5be173` · **Live:** 108,467 bytes

### 🌐 UI strings i18n Phase 4 — Geolocation templates
- **Extend `t()` function**: 支援 params — `t(key, { name, code, dist })` with `{name}` placeholder replacement, 向後兼容 `t(key)` 唔帶 params
- **14 個新 geolocation dict keys** (zh + en):
  - `geoSelected` / `geoLocating` / `geoLocatingWithAcc` (定位中 + 精度)
  - `geoAccuracyLow` / `geoAccuracyUnknown` (精度不足 / 未能確認)
  - `geoTooFar` (最近車站超出範圍)
  - `geoUncertainShow` / `geoUncertainKeep` (未能確定最近車站)
  - `geoLocated` (已定位至最近車站)
  - `geoGetFailed` / `geoNotSupported` / `geoPermissionDenied` (定位失敗)
  - `defaultStationMTR` / `defaultStationLRT` (預設車站 fallback)
- **17 個新 `t()` calls** 替換 `tryGeolocate()` / `setGeoNotice()` / `handleGeoFix()` 內嘅 template literals
- **`tryGeolocate()` function** 0 hardcoded Chinese remaining (was 19)
- **Comments in `handleGeoFix`** 保留中文 (developer-friendly)

### 📊 Stats
- Total `t()` calls: **42** (was 25)
- Total dict keys: **50** (was 36)
- 包含 9 個 geolocation templates + 12 個 static/UI strings

---

## v52.6.3 — 2026-10-04
**Commit:** `691f1a2` · **Live:** 106,359 bytes

### 🐛 CRITICAL FIX — `t()` function shadowed by local variable
- **Bug**: `appendDynamicJourney()` 入面 `const t = r.target` 遮蔽咗 global `t()` i18n helper
- **Symptom**: 上下行 計算失敗 `'t is not a function. (In "t('eta')", t is an instance of Object)'`
- **Fix**: Rename `const t = r.target` → `const tgt = r.target` (replace all `t.label` / `t.code` / `t.lines` with `tgt.*`)
- **Test**: TSH (Tai Shui Hang) 上下行 dynamic journey 應該正常顯示 預計抵達 + 車程 i18n strings

---

## v52.6.2 — 2026-10-04
**Commit:** `4f84bbf` · **Live:** 106,333 bytes

### 🌐 UI strings i18n Phase 2+3 (partial)
- **Phase 2 (textContent dynamic)**: offline/online notice → `t('offline')` / `t('online')`
- **Phase 3 (simple templates)**:
  - `minutesLabel`: 「即將到站」 / 「分鐘」 → `t('arriving')` / `t('minutes')`
  - `noService` / `noServiceData` / `platformNoTrips`
  - 「預計抵達」 / 「車程」 → `t('eta')` / `t('transit')`
  - 「月台」 + escapeHtml + 「暫無班次」 → `t('platform')` + `t('platformNoTrips')`
  - 「班次」 → `t('trips')`
  - 「支線：」 → `t('subLine')`
  - `stationNotFound` error message
- **25 個 `t()` calls** 全部 deploy ✅

---

## v52.6 — 2026-10-04
**Commit:** `093c07d` · **Live:** 106,149 bytes

### 🌐 UI strings i18n Phase 1 (static HTML)
- **`I18N_UI` dict** (33 keys zh + en, ~2.4KB JS)
- **`t(key)` helper function** + **`applyUIText()`** DOM walker
- **11 個 elements** marked with `data-i18n` attribute:
  - `appTitle` / `selectSystem` / `mtr` / `lrt` / `selectStation`
  - `locate` / `refresh` / `nextAutoUpdate` / `pause`
  - `disclaimer` / `privacy`
- **`runLang()`** 同步 call `applyUIText()` for textContent elements
- **DOMContentLoaded init** apply `currentLang` on load

---

## v52.5.1 — 2026-10-04
**Commit:** `4e2b1da` · **Live:** 102,464 bytes

### 🐛 Toggle bug fix
- **Bug**: `runLang()` 直接 call `refresh()` 但 missing `state.currentCode` / `state.currentLine` context, 所以 toggle 後 UI 唔即時 update
- **Fix**: 用 `selectStation(state.currentCode, state.currentLine)` (正確 entry point)
- **DOMContentLoaded init** apply saved lang (避免 reload 後返 zh)
- **Document title + H1** 跟住 toggle 切換

---

## v52.5 — 2026-10-04
**Commit:** `32ee135` · **Live:** 102,052 bytes

### 🌐 i18n toggle (中/EN) — Minimal scope
- **`STATION_NAMES_EN` map** (98 entries) — 從 [MTR official CSV](https://opendata.mtr.com.hk/data/mtr_lines_and_stations.csv) 擷取 EN station names
- **`stationName(code, line, lang)`** helper
- **`lineName(line)`** helper for LINE_NAMES i18n
- **`runLang()`** toggle zh ↔ en + localStorage persistence (`mtr-lang`)
- **Toggle button** top-right header (after Live indicator): `🌐 中/EN` / `🌐 EN/中`
- **LRT station names** 用 `LRT_STATIONS.name_en` (官方 CSV 已有 68 stops EN names)
- **Page title + H1** 跟住 lang 切換: 「港鐵實時列車到站資訊」 ↔ 「MTR Live Train Arrivals」

---

## v52.4 — 2026-10-04
**Commit:** `8235fc5` · **Live:** 97,754 bytes

### ✏️ UI text 重命名
- **Title / H1**: 「香港 MTR 實時列車到站資訊」 → **「港鐵實時列車到站資訊」** (移除冗餘「香港 MTR」, 因為成個 App 已經係香港 MTR)
- **車程 label**: 「相距」 → **「車程」** (更貼近用戶語感, 「相距」容易被誤會為「距離」)

### 🔒 Privacy notice (footer 新增)
- **新增** `<p>定位數據僅用於本機處理。</p>` — 說明定位只喺本機計算，唔會上傳 (純說明，唔係同意機制)

### 🛠️ Build infra
- v52.4.1: footer `id="app-version-footer"` 嘅 commit hash bump (`e8f2f5d` → `8235fc5`) — commit `d2ac0e0`

---

## v52.3 — 2026-10-02
**Commit:** `8355968` · **Merged:** PR #3 (`b039e2f`)

### 🛡️ Geolocation: more accurate, safer station pick
1. **Bug fix**: re-locate in auto mode only searched the system picked first (`state.mode === 'auto'` never true after `syncMode`) → uses `isAuto`
2. **Sampling**: `watchPosition` for up to 8 s, keep the most accurate fix, stop early at ≤ 50 m; watch always cleared; a manual pick cancels sampling
3. **Accuracy gate**: auto-select only when the gap to the 2nd-nearest station > 2 × accuracy; otherwise nearby-station chips
4. **Co-located MTR/LRT** (TUM/295, YUL/600, TIS/430 within 150 m): both offered, not counted as ambiguity
5. **Badge** shows accuracy, e.g. `📍 0.3 km · ±40 m`

Haversine itself is unchanged. Location stays on the device (not stored, logged or sent).

---

## v52.2 — 2026-10-02
**Commit:** `254ec63` (fixes `71d2922`) · **Merged:** PR #2 (`e8f2f5d`)

### 🔀 gh-pages → main
- `gh-pages` (v52.1) merged into `main` so GitHub Pages can deploy from `main` (Pages had been serving `gh-pages` while `main` stayed on v42/v43)

### 🛠️ Review fixes on top of v52.1
- TKL / EAL branches modelled as separate paths (`LINE_PATHS`); stops past a short-working train's destination no longer listed; branch cumulative minutes corrected
- Journey + Train Selector reuse the response already fetched (no extra API calls); stale responses dropped (`requestSeq`)
- Debug panel `$ is not a function` crash fixed; LRT → MTR "車站 undefined 不存在" fixed (`MTR::ADM` matched no option)
- Coordinates fixed: TSH, HEO, LOP, STW, TWH, LMC, SIH; LRT 250 = 屯門泳池; LRT routes +614P/705, −720/721/722

> PR #1 (`b108930`) had applied similar fixes to the old v42 `main` line (`8802fb6`, labelled v43); it never reached the live site and is superseded by v52.2.

---

## v52.1 — 2026-10-01
**Commit:** `29f4b07` · **Cache-bust:** `2af86bd` · **Live:** 89,068 bytes

### 🚉 屯馬綫 KSR 站
- **新增** TML 路線 MAJOR_STOPS: 將 **錦上路 KSR** 加入主要站列表
- TML_UP cum: WKS=0, KSR=58, YUL=62
- TML_DOWN cum: TUM=0, KSR=15

---

## v52 — 2026-09-30
**Commit:** `aa3220d`

### 🔧 P5 fixes (6 項)
1. Countdown-driven refresh only (removed the duplicate `refreshTimer`)
2. Auto mode = `userSelectedMode === null`
3. `bumpAutoRefreshAfterManual()` for refresh / retry / station change
4. Branch lines truncate at the branch point when `dest` is missing
5. Geolocation `maximumAge: 0` for user-initiated fixes
6. Removed unused `cdn.jsdelivr.net` preconnect

---

## v51.1 — 2026-09-30
**Commit:** `a61a213`

### 🎨 視覺修正
- Train Selector 揀中: `bg-cyan-700` → `bg-cyan-400` + `text-slate-900` (淺啲, 暗色文字 contrast 更好)

---

## v51 — 2026-09-30
**Commit:** `3d25948`

### 🛠️ VUL-09 fix
- 由 jsdelivr `tailwindcss@2.2.19` (缺 cyan/slate palettes) 改為本地 Tailwind 3.4 build (`assets/tailwind.css`)

---

## v10 – v50 (摘要, 按 commit 記錄)

- **v50.2** Train Selector 揀中: 移除 ✓ badge, 改用 `bg-cyan-700` + `border-cyan-300` (iOS Safari 穩定)
- **v50.1** Train Selector 揀中 visual: cyan bg + ring + ✓ badge
- **v50** 排版: 「選擇 系統」排喺「選擇車站」之前; 車站列表按 LINE_TOPOLOGY 列出全線全站 (允許重複, e.g. HUH 同時喺 EAL + TML)
- **v49.1** Upstream 空 payload 偵測 (e.g. EAL incident), 顯示「數據暫時無法取得」
- **v49** Train Selector: 揀 #1–#4 做動態行車時間 base train
- **v48** P4 code hardening: error boundary + fetch retry + stale guard + offline detection
- **v47.x** v47 P3 polish (mode / debug cyan, mobile stacked, error red); v47.1 LRT remark 只喺有值時顯示; v47.2 移除 inline raw debug blocks (+ footer label fix)
- **v46.x** padding / spacing 統一, 按鈕字縮短 (定位/刷新); v46.1–v46.2 PWA manifest cache-bust (之後 revert)
- **v45** P1: footer version stale fix + a11y (aria-label x8, aria-pressed, aria-live)
- **v44.x** 移除 sys/api/curr 時間顯示; raw debug 縮細並移到 footer
- **v43** Phase C: Tailwind Play CDN → pre-compiled CSS via jsdelivr (VUL-08)
- **v42** Phase B: branch-aware topology (TKL POA/LHP, EAL LOW/LMC), `userSelectedMode` decoupling, global AbortController (VUL-02/05/07)
- **v41** Phase A: SWH→西灣河, ERL→TCL + 移除 WRL, 合併 DOMContentLoaded, resetCountdownTimer (VUL-01/03/04/06)
- **v40** Raw API debug section
- **v36–v39** Auto 模式: 開 app 自動比較最近重鐵/輕鐵站; v39 移除自動按鈕 (auto 只係初始狀態)
- **v34–v35** LRT_STATIONS 68 站, 按官方 LR Next Train Data Dictionary v1.2 核對
- **v31–v33** 定位重構: isValidCoordinate, 最近站 reduce, 4 道關卡 (座標 → 精度 ≤1000 m → 有站 → ≤2.5 km)
- **v20–v30** 動態行車時間改用靜態累計分鐘 (CUMULATIVE_MINUTES), MAJOR_STOPS, TML 雙向不對稱表, DOWN 方向計算修正
- **v10–v19** 按官方 Next Train API spec v1.7 建立 98 個站碼, STATIONS / LINE_TOPOLOGY

---

## 🔒 Security posture

mtr-app **冇後端**, 所有 API calls 喺 client-side 直接 call `https://rt.data.gov.hk/` (香港政府公開資料 API)。

| Aspect | Status |
|---|---|
| API key 暴露 | ✅ None |
| PII 暴露 | ✅ None |
| HTTPS-only | ✅ Yes |
| eval / XSS vector | ✅ None |
| Cookies / localStorage | ✅ None (除咗 language/theme preference；唔儲存位置) |
| Geolocation | ✅ Opt-in only |
| Backend | ✅ 完全無 |

---

## 📜 License / Data attribution

- 數據來源: [data.gov.hk MTR Next Train API](https://data.gov.hk/en-data/dataset/mtr-data2-nexttrain-data)
- API spec: [Next Train API Spec v1.7](https://opendata.mtr.com.hk/doc/Next_Train_API_Spec_v1.7.pdf)
- LRT spec: [LR Next Train Data Dictionary v1.2](https://opendata.mtr.com.hk/doc/LR_Next_Train_DataDictionary_v1.2.pdf)

---

*Last updated: 2026-10-04 · commit `a0f1bea` (v52.7.4)*