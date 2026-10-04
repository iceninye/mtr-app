# 港鐵到站 — Changelog

> MTR Next Train PWA · https://iceninye.github.io/mtr-app/
> 全部 client-side, 冇 backend, 數據來源: [data.gov.hk MTR Next Train API](https://data.gov.hk/en-data/dataset/mtr-data2-nexttrain-data)

---

## v52.4 — 2026-10-03
**Commit:** `8235fc5` · **Live:** 97,754 bytes

### ✏️ UI text 重命名
- **Title / H1**: 「香港 MTR 實時列車到站資訊」 → **「港鐵實時列車到站資訊」** (移除冗餘「香港 MTR」, 因為成個 App 已經係香港 MTR)
- **車程 label**: 「相距」 → **「車程」** (更貼近用戶語感, 「相距」容易被誤會為「距離」)

### 🔒 Privacy notice (footer 新增)
- **新增** `<p>定位數據僅用於本機處理。</p>` — 對應香港 PDPO (個人資料條例) 用戶知情同意要求, 配合 navigator.geolocation.opt-in

### 🛠️ Build infra
- v52.4.1: footer `id="app-version-footer"` 嘅 commit hash bump (`e8f2f5d` → `8235fc5`) — commit `d2ac0e0`

---

## v52.3 — 2026-10-02
**Commit:** `8355968` · **Live:** 97,701 bytes

### 🛡️ Geolocation 改進
- 「more accurate, safer geolocation station pick」
- 🎯 **更精準** — Haversine distance 計算 refine, 用戶位置 → 最近車站配對更準
- 🛡️ **更安全** — `navigator.geolocation.watchPosition` 嘅 error handling 改進 (timeout, permission denied fallback)

### 📋 合併記錄
- PR #1 (b108930): v43 code review fixes
- PR #2 (e8f2f5d): 2026-09-30 cache-bust merge
- PR #3 (b039e2f): v52.3 final merge

---

## v52.2 — 2026-10-02 (中)
**Commit:** `8802fb6` · **Live:** (intermediate)

### 🛠️ v43 code review fixes
- 多個 P0-P3 code review issues resolution
- 細節見 commit diff

---

## v52.1 — 2026-10-01
**Commit:** `29f4b07` · **Live:** 89,068 bytes · **Cache-bust:** `2af86bd`

### 🚉 屯馬綫 KSR 站
- **新增** TML 路線 MAJOR_STOPS: 將 **錦上路 KSR** 加入主要站列表
- TML_UP cum: WKS=0, KSR=58, YUL=62
- TML_DOWN cum: TUM=0, KSR=15

### 🔧 P5 fixes (6 項)
1. **Accessibility**: `aria-label` x8 + `aria-pressed` + `aria-live`
2. **Cache-bust**: CDN cache invalidation via `2af86bd` commit
3. **Performance**: 移除 `cdn.jsdelivr.net` preconnect (P5 #6)
4. **Geolocation**: `maximumAge: 0` for user-initiated (P5 #5)
5. **UI**: status icon ✅ ⚠️ consistency
6. **Background**: 改用 Tailwind pre-compiled CSS 取代 JIT Play CDN

---

## v52 — 2026-09-30
**Commit:** `aa3220d` · **Live:** (pre-KSR)

### 🔧 P5 fixes (6 項, 沿用)
- 同上 v52.1 P5 fixes 1-6

---

## v51.1 — 2026-09-30
**Commit:** `a61a213`

### 🎨 視覺修正
- `bg-cyan-400` 取代 `bg-blue-500` — iOS-compatible cyan

---

## v51 — 2026-09-29

### 🛠️ VUL-09 fix
- Local Tailwind build (取代 JIT Play CDN) — 防止 `play.tailwindcss.com` 嘅 CDN dependency + 第三方供應鏈攻擊面

---

## v50.x — 2026-09 (歷次)

### 🚂 v50.2 — iOS-compatible basic cyan
### 🔘 v50.1 — Train Selector 揀中 visual (border highlight)
### ↕️ v50 — DOM reorder (Train Selector 置頂)
### 🏷️ v49.1 — EAL upstream banner (東鐵綫實時班次 upstream 暫無回應提示)
### 🚉 v49 — Train Selector (揀邊卡車做 reference train)
### 🛡️ v48 — P4 Code Hardening (SQL injection prevention, type checks, edge cases)
### 🌐 v47.3 — Footer label fix
### 🛠️ v47.2 — Raw debug 移除
### 🚉 v47.1 — LRT remark suppress
### ♿ v47 — P3 (小 fixes)
### ♿ v45 — P1 accessibility (首次 a11y pass)
### 🔒 v43 — Code review (8 個 VUL audit fixes)
### 🚉 v42 — Auto-mode (auto-detect MTR vs LRT based on selected station)
### 🚉 v41 — 定位重構 (Haversine formula fix)
### 🚉 v40 — LRT_STATIONS 完整 68 站座標庫
### 🚉 v30 — 全線全站
### 🚉 v20 — 同路線站間動態行車時間
### 🚉 v10 — MTR/LRT PWA 初版

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

## v52.6 — 2026-10-03
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

## v52.5.1 — 2026-10-03
**Commit:** `4e2b1da` · **Live:** 102,464 bytes

### 🐛 Toggle bug fix
- **Bug**: `runLang()` 直接 call `refresh()` 但 missing `state.currentCode` / `state.currentLine` context, 所以 toggle 後 UI 唔即時 update
- **Fix**: 用 `selectStation(state.currentCode, state.currentLine)` (正確 entry point)
- **DOMContentLoaded init** apply saved lang (避免 reload 後返 zh)
- **Document title + H1** 跟住 toggle 切換

---

## v52.5 — 2026-10-03
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

## 🔒 Security posture

mtr-app **冇後端**, 所有 API calls 喺 client-side 直接 call `https://rt.data.gov.hk/` (香港政府公開資料 API)。

| Aspect | Status |
|---|---|
| API key 暴露 | ✅ None |
| PII 暴露 | ✅ None |
| HTTPS-only | ✅ Yes |
| eval / XSS vector | ✅ None |
| Cookies / localStorage | ✅ None (除咗 language/theme preference) |
| Geolocation | ✅ Opt-in only |
| Backend | ✅ 完全無 |

---

## 📜 License / Data attribution

- 數據來源: [data.gov.hk MTR Next Train API](https://data.gov.hk/en-data/dataset/mtr-data2-nexttrain-data)
- API spec: [Next Train API Spec v1.7](https://opendata.mtr.com.hk/doc/Next_Train_API_Spec_v1.7.pdf)
- LRT spec: [LR Next Train Data Dictionary v1.2](https://opendata.mtr.com.hk/doc/LR_Next_Train_DataDictionary_v1.2.pdf)

---

*Last updated: 2026-10-04 · commit `aed8ab4` (v52.7.1)*