# ✈ Travel Note 模板

純前端旅遊行程規劃模板，無需後端，部署成本為零。基於 [japantravel-six.vercel.app](https://japantravel-six.vercel.app/itinerary.html) 的設計重構，抽掉個人資料、加上完整中文註解，作為可重複使用的範本。

## 功能

- 🗓 **行程時間軸**：每日活動清單，可線上編輯、拖曳排序、Undo
- 🗺 **互動地圖**：Leaflet + OpenStreetMap，免 API key、免註冊
- ☀ **天氣預報**：Open-Meteo API，免 API key、免註冊；失敗時 fallback 為歷史平均
- 💾 **資料儲存**：瀏覽器 localStorage，每位訪客獨立（編輯不影響其他人）
- 📱 **RWD**：手機桌機都能用
- 🎨 **手寫風美感**：Klee One + Noto Serif TC，米色＋橘色配色

## 技術組成

| 項目 | 技術 |
|---|---|
| 前端 | 純 vanilla HTML/CSS/JS（無 framework、無 build step） |
| 地圖 | Leaflet 1.9.4 (CDN) |
| 字體 | Google Fonts (Noto Serif TC + Klee One) |
| 天氣 | [Open-Meteo](https://open-meteo.com/) Forecast API |
| 儲存 | `localStorage`（key 由 `CONFIG.storageKey` 決定） |
| 拖曳 | HTML5 原生 Drag & Drop API |
| 部署 | 任何靜態 hosting（Vercel / Netlify / Cloudflare Pages / GitHub Pages） |

整個專案就一個 `index.html`，沒有 build、沒有 npm install、沒有 node_modules。

## 快速開始

### 1. 本機預覽

直接雙擊 `index.html` 在瀏覽器打開即可。
> ⚠ 注意：用 `file://` 開啟時，天氣 API 可能因 CORS 政策被擋（會自動 fallback 到歷史平均）。部署到任何 https 網域後就會正常。

### 2. 部署到 Vercel（推薦）

見下方 [部署到 Vercel](#部署到-vercel) 區塊。

## 客製化

只需修改 `index.html` 裡的兩個區塊：

### A. CONFIG 設定區（檔案中段）

```js
const CONFIG = {
  pageTitle: '✈ Travel Note · 京都櫻花',         // 瀏覽器分頁標題
  headerSubtitle: '2026 · 京都櫻花',               // logo 旁的副標題
  storageKey: 'tn_kyoto_2026',                     // localStorage key（不同專案用不同名稱）

  weatherStartDate: '2026-04-01',                  // 旅程起日（YYYY-MM-DD）
  weatherEndDate:   '2026-04-03',                  // 旅程迄日

  weatherLatitude:  35.0116,                       // 目的地中心座標（用於天氣 API）
  weatherLongitude: 135.7681,

  dayColors: ['#E8642A', '#5a8a5a', '#2a6a9a', '#c8860a', '#7b5ea7'],

  weatherFallback: [                               // 天氣 API 失敗時用的歷史平均
    { date:'4/1', day:'Day 1', icon:'🌸', high:18, low:9, desc:'晴', rain:'降雨機率 20%' },
    // ...
  ],
};
```

> ⚠ Open-Meteo 只能查詢過去與未來約 16 天內的天氣。出發日太遠時會失敗，自動使用 `weatherFallback`。

### B. 預設行程資料

#### `SPOTS`（景點座標表）

```js
const SPOTS = {
  '清水寺': { lat: 34.9949, lng: 135.7851, hint: '京都最古老寺院之一' },
  '%Arabica 嵐山': { lat: 35.0136, lng: 135.6770, hint: '人氣咖啡廳' },
};
```

- key 必須跟 `DEFAULT_ITINERARY` 裡某個 activity 的 `name` 完全一致，地圖才會顯示對應標記
- 取得座標：Google Maps 上對景點按右鍵，第一行就是經緯度（點一下會複製）
- `hint` 是地圖 popup 的補充說明，可省略

#### `DEFAULT_ITINERARY`（預設行程）

```js
const DEFAULT_ITINERARY = [
  { id:'d1', day:'Day 1', date:'4/1 (三)', region:'京都市區', icon:'🌸', activities:[
    { id:'a1', icon:'✈️', name:'抵達關西機場', sub:'CI 156｜桃園 → 關西 09:30' },
    { id:'a3', icon:'⛩',  name:'清水寺', sub:'下午散步、和服體驗',
      url:'https://www.kiyomizudera.or.jp/', urlLabel:'🌐 官網' },
  ]},
  // ...
];
```

| 欄位 | 必填 | 說明 |
|---|---|---|
| `id` | ✅ | 唯一識別碼，可用 `'d1'`/`'a1'` 等短碼 |
| `day` | ✅ | 第幾天（會顯示在標題） |
| `date` | ✅ | 日期字串，自由格式 |
| `region` | ✅ | 區域，顯示在標題下方 |
| `icon` | ✅ | 該日 emoji |
| `activities` | ✅ | 活動陣列 |

每個 activity：

| 欄位 | 必填 | 說明 |
|---|---|---|
| `id` | ✅ | 唯一識別碼 |
| `icon` | ✅ | emoji |
| `name` | ✅ | 活動名稱（若與 SPOTS key 相符會出現在地圖） |
| `sub` |   | 備註說明 |
| `url` + `urlLabel` |   | 單一外部連結 |
| `urls` |   | 多個外部連結，格式 `[{href, label}, ...]` |

### C. 配色主題（選配）

如果想換整體配色，修改檔案最上面的 CSS 變數：

```css
:root{
  --orange:#E8642A;     /* 主色 */
  --cream:#FAF7F2;      /* 背景 */
  --ink:#1C1008;        /* 主要文字 */
  /* ... */
}
```

## 部署到 Vercel

### 一次性設定

1. **登入 Vercel**：[vercel.com](https://vercel.com) → Sign up with GitHub
2. **Import 專案**：右上 **Add New → Project** → 選你的 repo（會自動列出）
3. **Framework Preset** 選 **Other**（純靜態，不要選 Next.js）
4. **Build Command** 留空（不需要 build）
5. **Output Directory** 留空（預設就是 repo 根目錄）
6. 點 **Deploy** → 約 30 秒後拿到 `xxx.vercel.app` 網址

### 之後的更新

在本機 `git push` 到 GitHub 後，Vercel 會自動偵測並重新部署，約 20 秒就會上線。

### 自訂網域（選配）

Vercel 專案設定 → **Domains** → 加入你的網域 → 依指示在 DNS 設一筆 CNAME 指向 `cname.vercel-dns.com`。SSL 憑證 Vercel 會自動申請。

## 編輯按鈕

頁面右上角有「✏️ 編輯」按鈕，點擊後可：

- 修改每天的日期、地區、emoji
- 新增/刪除日子
- 新增/刪除/排序活動
- 拖曳 ⠿ 把手調整活動順序
- 「↩ 上一步」回復誤操作（最多 30 步）
- 「✓ 儲存」才會真正寫入

> ⚠ 修改只存在當前瀏覽器的 localStorage，**不會同步到伺服器**，也不會影響其他訪客。如果想永久變更預設內容，請直接修改 `DEFAULT_ITINERARY` 並重新部署。
>
> 不需要編輯功能的話，刪除 HTML 中那顆 `<button class="btn-edit">` 即可。

## 重設行程

如果想清除 localStorage 裡的編輯紀錄，恢復成程式碼裡的預設行程：

開啟瀏覽器 DevTools (F12) → Console 輸入：

```js
localStorage.removeItem('tn_kyoto_2026');  // 換成你 CONFIG.storageKey 的值
location.reload();
```

## 已知限制

- 編輯資料只存在當前瀏覽器（localStorage），不同裝置間不會同步
- 跨天拖曳尚未支援（只能同一天內排序）
- Open-Meteo 不能查詢太遠的未來日期（超過 16 天會 fallback）
- 景點座標必須手動填寫到 `SPOTS`（沒有自動 geocoding）

## 授權

模板本身可自由使用、修改、再散布。內含資源：
- Leaflet — BSD-2-Clause
- Open-Meteo — CC BY 4.0
- OpenStreetMap — ODbL
- Google Fonts — SIL Open Font License

## 設計來源

設計參考自 [japantravel-six.vercel.app](https://japantravel-six.vercel.app/itinerary.html)。
