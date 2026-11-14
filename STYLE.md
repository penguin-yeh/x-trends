# X 熱門話題頁面風格規範：「夜間時間軸」

> 給負責產生本 repo 頁面的 bot（Grok bot）：**產生或更新任何 HTML 之前，先完整讀完這份文件並嚴格遵守。**
>
> 這份規範是在**現有頁面的基礎上客製化**：保留現在的 X 風格時間軸（深色、三欄、貼文卡片、分頁、篩選膠囊、右側「現正流行」），不推翻重做。新增的重點有三個：**分類色、有圖就附圖、「新」標記改色**。沒有圖的貼文完全沒有關係，照常顯示。

## 0. 硬性規則

1. 單檔自含：CSS、JS 寫在頁面裡，不載外部字型、CSS、CDN。允許單一 `<style>` 區塊（需要 `:hover`、媒體查詢與 CSS 變數）。**唯一的外部資源是貼文附圖**（見第 4 節）。
2. 深色單一主題，維持 `<meta name="color-scheme" content="dark">`，不做淺色版。
3. 不准：漸層、`box-shadow`、動畫位移；`backdrop-filter` 只准用在頂部 sticky 列（現有做法）。
4. 版面維持現狀：`260px / 600px / 350px` 三欄；寬度不足時依現有媒體查詢收成兩欄、單欄。手機（約 375px）不可橫向捲動。
5. 所有文字繁體中文。連結、時間、數字必須對應原始來源，不可編造。
6. 外部連結一律 `rel="noopener noreferrer"`。

## 1. 設計 token

在 `:root` 定義，元件裡不要直接寫色碼（分類色除外，見第 2 節）：

```css
:root{
  --bg:#0a0c0f;      /* 比純黑柔一點，圖片與文字才有層次 */
  --panel:#13171c;
  --hover:#0e1115;
  --line:#262c33;
  --ink:#e7e9ea;
  --mute:#7a8590;
  --blue:#1d9bf0;    /* 連結、目前分頁底線 */
  --new:#f5b84a;     /* 「新」標記專用琥珀色 */
  --new-ink:#1a1200;
  --mono:ui-monospace,SFMono-Regular,Consolas,monospace;
}
```

與舊版的差異：背景從 `#000` 改 `#0a0c0f`；「新」badge 從藍色改琥珀色（藍色留給連結與目前狀態，兩者不再混淆）。

## 2. 分類色

每則貼文的分類（`.av` 的 `title`）對應一個顏色，只用在三處：頭像圓的底色、頭像旁分類名稱（`.hd b`）的文字色、右側細線。**不要拿來填滿大面積。**

| 分類 | 色 |
|---|---|
| 資安 | `#ff6b6b` |
| 產業、財經、晶片、科技 | `#f5b84a` |
| AI | `#8b93ff` |
| 開發、開發工具、開源、雲端 | `#3ecf8e` |
| 國際、國際政治、政治、政策、社會、台灣 | `#4cc9f0` |
| 體育、遊戲 | `#ff9f43` |
| 娛樂、音樂、迷因、生活、天氣、健康 | `#ff7ac6` |
| 其他（未列出） | `#7a8590` |

做法：貼文根元素帶 `style="--cat:#ff6b6b"`，CSS 取用：

```css
.av{background:color-mix(in srgb,var(--cat,#7a8590) 24%,var(--bg));border:1px solid color-mix(in srgb,var(--cat,#7a8590) 55%,var(--bg))}
.hd b{color:var(--cat,var(--ink))}
```

頭像裡的 emoji 保留（它是資料的一部分），一則貼文一個。

## 3. 元件寫法

保留現有 class 名稱，只改樣式與新增媒體區塊。

### 3.1 貼文卡片

```css
.post{display:flex;gap:12px;padding:14px 16px;border-bottom:1px solid var(--line)}
.post:hover{background:var(--hover)}
.hd{display:flex;flex-wrap:wrap;align-items:center;gap:4px 6px;font-size:.94rem;line-height:1.3}
.hd .g{color:var(--mute);font-family:var(--mono);font-size:.78rem}
.badge{background:var(--new);color:var(--new-ink);font-size:.68rem;font-weight:800;border-radius:4px;padding:1px 6px}
.pt{font-weight:700;font-size:1.04rem;line-height:1.45;margin:6px 0 4px}
.ps{margin:0;font-size:.95rem;line-height:1.6;color:#d6d9db}
```

時間與「每小時新增」這類小字用等寬字（`.hd .g`），其他維持無襯線。

### 3.2 附圖區塊（本次新增）

圖片放在摘要（`.ps`）與操作列（`.acts`）之間。結構：

```html
<figure class="media">
  <img src="圖片網址" alt="貼文標題" loading="lazy" referrerpolicy="no-referrer"
       onerror="this.closest('.media').remove()">
  <figcaption><span class="dom">來源網域</span></figcaption>
</figure>
```

```css
.media{margin:10px 0 0;border:1px solid var(--line);border-radius:16px;overflow:hidden;background:var(--panel)}
.media img{display:block;width:100%;max-height:300px;object-fit:cover}
.media figcaption{padding:6px 12px;border-top:1px solid var(--line);font:.74rem var(--mono);color:var(--mute)}
```

- `onerror` 讓載入失敗的圖整塊消失，不留破圖框。
- `figcaption` 只放來源網域（例如 `anthropic.com`），不放長文字。
- 整張圖不需要可點；外連入口維持現有的「在 X 上查看 ↗」按鈕。

### 3.3 其餘區塊（維持現狀，只換 token）

置頂「今日重點」、分頁 `.tabs`、篩選膠囊 `.fbar`、右側 `.rbox` 與 `.tr`、頁尾 `.foot` 的結構都不變，顏色改用第 1 節的 token。分頁底線與連結仍用 `--blue`。

## 4. 圖片規則

**有圖就附，沒有圖就不放，完全沒關係。** 不要為了湊版面而補圖。

來源優先順序（依序找，找到第一個就停）：

1. 該話題或貼文本身附帶的圖片（X 貼文的媒體）。
2. 貼文摘要所依據的原始新聞或文章頁面的 `og:image`（找不到再試 `twitter:image`）。
3. 都沒有 → **不放 `<figure class="media">`**，整塊省略。

硬性限制：

- 只用 `https://` 網址，且抓取時 HTTP 200、`Content-Type` 為 `image/*`；任一條件不符就當沒有圖。
- **不准**：網站 logo／favicon 充當圖片、別篇文章的圖、AI 生成圖、佔位圖、自己編造的網址、對圖片做任何下載後再上傳到別處的處理。
- 一則貼文最多一張圖。
- 圖片只引用原網址（hotlink），不要把圖片內容用 base64 內嵌進 HTML（頁面已經很大）。
- 一定要 `loading="lazy"`，因為單一頁面內有數百則貼文。
- 涉及人物、事故現場等敏感畫面時，寧可省略也不要附。

## 5. 檢查清單

- [ ] `:root` 使用第 1 節 token，沒有 `#000` 純黑背景、沒有漸層與陰影。
- [ ] 每則貼文都有 `--cat`，分類色只用在頭像圓、分類名稱與細線。
- [ ] 「新」標記是琥珀色，不是藍色。
- [ ] 有附圖的貼文：`<img>` 帶 `loading="lazy"`、`alt`、`onerror`；網址是 https 且確實是圖片。
- [ ] 沒有圖的貼文**沒有**空的 `.media` 區塊或佔位圖。
- [ ] 375px 寬度下無橫向捲動；三欄在寬螢幕仍是 260/600/350。
- [ ] 日期切換、分頁切換、篩選膠囊仍可用。
