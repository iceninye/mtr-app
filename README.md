# 🚇 港鐵到站 · MTR Next Train

港鐵重鐵 + 輕鐵實時到站資訊 PWA。開 app 自動搵最近嘅車站，顯示下幾班車幾時到。全部喺瀏覽器 client-side 運行，冇 backend。

**▶ 立即使用：** https://iceninye.github.io/mtr-app/
（Cloudflare 鏡像：https://mtr-app.iceninye.workers.dev）

---

## ✨ 功能

- **重鐵 10 條綫 + 輕鐵 68 個站**：港島綫、荃灣綫、觀塘綫、將軍澳綫、東涌綫、屯馬綫、東鐵綫、南港島綫、迪士尼綫、機場快綫，以及輕鐵全部車站。
- **自動定位最近車站**：開 app 時比較最近嘅重鐵站同輕鐵站，自動揀最近嗰個；附近仲有其他站就以 chips 列出，一撳即轉。重鐵 ↔ 輕鐵同一位置嘅站（屯門、元朗、天水圍、兆康）會互相提示。定位只喺本機處理，唔會上傳或儲存。
- **實時到站**：每 12 秒自動更新，顯示上行 / 下行頭兩班車（可展開睇更多）、月台、目的地；東鐵綫開出時間會標明「↑ 開出時間」。
- **動態行車時間**：揀一班車做基準，估算沿綫主要車站幾時到。
- **非服務時間 / 故障提示**：夜晚收車、API 冇回應、上游數據暫停時，會顯示該站嘅提示卡同實際原因（例如 `HTTP 503`），唔會留低上一個站嘅舊資料。
- **中 / EN 切換**、**深色 / 淺色 / 跟系統** 主題（偏好存喺 `localStorage`）。
- **PWA**：可以「加至主畫面」，iOS / Android 以全螢幕 app 形式開啟。
- **Debug 面板**：定位狀態、Raw API metadata、原始列車 JSON，方便排查問題。

## 📡 數據來源

| 用途 | 來源 |
|---|---|
| 重鐵到站 | [data.gov.hk MTR Next Train API](https://data.gov.hk/en-data/dataset/mtr-data2-nexttrain-data)（`rt.data.gov.hk/v1/transport/mtr/getSchedule.php`，[API Spec v1.7](https://opendata.mtr.com.hk/doc/Next_Train_API_Spec_v1.7.pdf)） |
| 輕鐵到站 | `rt.data.gov.hk/v1/transport/mtr/lrt/getSchedule`（[LR Next Train Data Dictionary v1.2](https://opendata.mtr.com.hk/doc/LR_Next_Train_DataDictionary_v1.2.pdf)） |
| 車站座標 | [hkbus/hk-bus-crawling](https://github.com/hkbus/hk-bus-crawling) `stopList`（由 data.gov.hk / 港鐵開放數據整合），站名按 [港鐵輕鐵路綫及車站 CSV](https://opendata.mtr.com.hk/data/light_rail_routes_and_stops.csv) 核對 |

> 本工具只供參考，請以車站現場廣播為準。本項目與港鐵公司無關。

## 🗂️ 檔案結構

```
index.html              整個 app（HTML + JS，車站資料、i18n、定位、API 都喺入面）
assets/tailwind.css     預先編譯嘅 Tailwind CSS（由 tailwind.src.css 生成）
assets/tailwind.src.css Tailwind 原始檔
assets/icons/           PWA icons
manifest.webmanifest    PWA manifest
tailwind.config.js      Tailwind 設定
wrangler.jsonc          Cloudflare Workers 靜態資源設定（冇 Worker script）
.assetsignore           唔上載去 Cloudflare 嘅檔案
CHANGELOG.md            每個版本嘅改動、驗證結果
```

## 🛠️ 本機運行

純靜態網站，唔使 build。喺 repo 根目錄開個 HTTP server：

```bash
python3 -m http.server 8000
# 打開 http://localhost:8000
```

定位功能需要 HTTPS 或 `localhost`。

如果改咗 `index.html` 入面用到嘅 Tailwind class，要重新編譯 CSS：

```bash
npx tailwindcss@3.4 -i assets/tailwind.src.css -o assets/tailwind.css --minify
```

## 🚀 部署

`main` 分支有新 commit 就會自動部署去兩個地方：

- **GitHub Pages** → https://iceninye.github.io/mtr-app/
- **Cloudflare Workers Builds**（`npx wrangler deploy`，按 `wrangler.jsonc` 將 repo 根目錄當靜態資源上載）→ https://mtr-app.iceninye.workers.dev

## 🔢 版本同 Changelog

版本號顯示喺頁面 footer（例如 `v52.9.0 · commit 9012dcb`）。改 app 嘅慣例：

1. 第一個 commit：改動 + 版本號 bump。
2. 第二個 commit：將第一個 commit 嘅 hash 寫入 footer 同 [`CHANGELOG.md`](CHANGELOG.md)。

每個版本嘅問題、修正同瀏覽器驗證結果都記錄喺 [CHANGELOG.md](CHANGELOG.md)。

## 🔒 私隱同安全

- 冇後端、冇 API key、冇 cookies。
- 所有 API call 由瀏覽器直接 call `https://rt.data.gov.hk/`。
- 定位要用戶允許先會用，只喺本機計算最近車站，唔會儲存。
- `localStorage` 只儲存語言、主題偏好，同邊啲區塊（例如同路線站間動態行車時間、輕鐵月台）收起咗。
