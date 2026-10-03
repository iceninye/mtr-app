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

*Last updated: 2026-10-03 · commit `d2ac0e0`*