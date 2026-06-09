# 📋 中文範文練習卷生成器 - 開發 Prompt

## 我嘅身份
我係香港中學中文補習老師，教 DSE 中文 Paper 1 甲部 12 篇範文，喺幾間唔同學校教書，每間進度唔同。我**唔識寫 code**，需要極簡操作。

## 要解決嘅問題
每間學校教到唔同範文、唔同進度，要不停手動度身做唔同嘅練習卷。需要：勾選範圍 → 設定每類題目嘅抽題數量 → 隨機抽題 → 印 PDF 俾學生做。

## 技術要求
- **單一 HTML 檔案**（無 build step）
- **Firebase Auth (Google) + Firestore** 跨裝置同步
- **GitHub Pages** 免費 hosting
- **手機友善**（sticky header、大掣）
- **完全免費**

## Firebase config
```js
const FIREBASE_CONFIG = {
  apiKey: "AIzaSyA_jHKrdxPApu42xHvy7on0HGR80kQk9fI",
  authDomain: "dse-chinese-99378.firebaseapp.com",
  projectId: "dse-chinese-99378",
  storageBucket: "dse-chinese-99378.firebasestorage.app",
  messagingSenderId: "1074279400925",
  appId: "1:1074279400925:web:07b61ff15d1a0f3c456234"
};
```
Firestore：`users/{uid}/data/main`，桌面 `signInWithPopup`、手機 `signInWithRedirect`，IndexedDB persistence。

## 8 種題型 + 分數規則
| key | 顯示 | 分數規則 |
|---|---|---|
| `translate` | 語譯 | **固定 1 分／題（強制）** |
| `judge` | 三式判斷題 | **固定 1 分／題（強制）** |
| `mc` | 多項選擇題（單選）| **固定 2 分／題（強制）** |
| `multi_mc` | 多選題（可多選）| 每題自定（必填）|
| `short` | 單答題 | 每題自定（必填）|
| `long` | 複答題 | 每題自定（必填）|
| `table` | 表格填寫題 | 每題自定（必填）|
| `composite` | 複合題（多分項）| 每題自定 = sub 總和 |

## 16 篇預設範文（詩詞拆開）
1. 論仁、論孝、論君子（《論語》）
2. 魚我所欲也（《孟子》）
3. 逍遙遊（《莊子》）
4. 勸學（荀子）
5. 廉頗藺相如列傳（司馬遷）
6. 出師表（諸葛亮）
7. 師說（韓愈）
8. 始得西山宴遊記（柳宗元）
9. 岳陽樓記（范仲淹）
10. 六國論(蘇洵)
11. 山居秋暝（王維）
12. 月下獨酌（其一）（李白）
13. 登樓（杜甫）
14. 念奴嬌・赤壁懷古（蘇軾）
15. 聲聲慢・秋情（李清照）
16. 青玉案・元夕（辛棄疾）

（**無「唐詩三首」「宋詞三首」合集**）

## 資料結構

### 單題（translate / judge / mc / multi_mc / short / long）
```json
{
  "id": "uuid",
  "source": "師說（韓愈）",
  "origin": "2023 DSE 練習卷",
  "type": "translate|judge|mc|multi_mc|short|long",
  "text": "題目內容",
  "answer": "答案 或 array（multi_mc）",
  "marks": 1,
  "options": {"A":"...","B":"...","C":"...","D":"..."}
}
```

**`source` 欄位特殊規則**：
- 一般單篇題目：string，例如 `"source": "師說（韓愈）"`
- 跨篇章題目（同時涉及兩篇或以上範文）：array，例如 `"source": ["魚我所欲也（《孟子》）", "師說（韓愈）"]`
- 內部統一處理：當作 array 處理（string 自動 normalize 做單元素 array）
- 篩選 / 檢索：揀某篇章時，題目 source array 入面**任何一個**配中，即計入結果
- 顯示：題目 tag 上面顯示用「、」分隔多篇

### 表格題
```json
{
  "id": "uuid",
  "source": "岳陽樓記（范仲淹）",
  "origin": "教科書出版社題庫",
  "type": "table",
  "text": "題目指示（例：根據引文填寫下表）",
  "marks": 4,
  "headers": ["", "行道之人", "乞人"],
  "rows": [
    ["遭遇", "____", "____"],
    ["反應", "____", "____"]
  ],
  "answer": "遭遇：(1) ⋯ (2) ⋯ ／反應：(1) ⋯ (2) ⋯"
}
```
- `____`（4 個下劃線）= 學生要填**整格**嘅空格 → render 成空白 cell（最低高度 50pt）
- `[填]`（一對方括號加「填」字）= 一格 cell 入面**部分文字部分要填**嘅 inline 標記 → render 成 inline 橫線（例如 `_______` 一段）
- `headers` 同 `rows` 每行欄數要一致
- 表格如有「行標題」（例如 遭遇／反應），headers 第一格放 `""`，每 row 第一格放行標題文字（**唔係 blank**）

### 複合題
```json
{
  "id": "uuid",
  "source": "...",
  "origin": "...",
  "type": "composite",
  "text": "題莖（共用引文／描述）",
  "marks": 6,
  "sub_questions": [
    {"label":"(1)","text":"...","answer":"...","marks":2,"sub_type":"short"},
    {"label":"(2)","text":"...","answer":"...","marks":4,"sub_type":"long"}
  ]
}
```
sub_type 可以係除 composite 以外任何類型（包括 table、mc、multi_mc）。

## 「來源」(origin) 欄位
- **設定分頁**有「來源管理」section：手動新增 / 刪除
- 批量輸入頁頂部 dropdown「呢批題目嘅來源」（可揀或打字新增），確認時整批套用
- 單獨新增 / 編輯題目時都可揀
- 出卷篩選、檢索可按來源過濾

## 4 個分頁
1. **題庫管理**（含檢索）
2. **批量輸入**
3. **出卷**（含 PDF 預覽 + 種子碼）
4. **設定**（範文管理／來源管理／登入登出／匯出匯入）

---

## 🔥 六大功能重點

### ① 印卷版面（極重要）

**字體要求**：
```css
.paper-print, .paper-print * {
  font-family: "PMingLiU","新細明體",serif !important;
  font-size: 12pt !important;
  color: #000 !important;
}
```
全文字**新細明體 12pt 純黑**，唔好用灰、藍、淺色。

**分頁規則**（極重要）：
每條題目 + 答題空間必須**同一頁**，唔夠位就成條推到下一頁。  
- Composite **連所有 sub_questions 必須同一頁**
- Table **整個表格必須同一頁**
```css
.paper-question, .paper-composite, .paper-table-q {
  page-break-inside: avoid;
  break-inside: avoid;
}
.paper-section h2 {
  page-break-after: avoid;
  break-after: avoid;
}
```

**列印時只顯示**：卷頭（科目／日期／姓名／班別／分數欄／可選種子碼）、題目、答題空間。  
**隱藏**：navigation、所有按鈕、設定 UI、答案、source/origin tag、檢索欄。用 `@media print` 全部 hide。

**答題留白**：
| 題型 | 留白 |
|---|---|
| 語譯字解 | 1 行 |
| 語譯句譯（「試語譯」開頭）| 2 行 |
| 三式判斷 | 1 行（圈 ○／✗／？）|
| MC | 1 行（圈 A/B/C/D）|
| 多選 MC | 1 行（圈多個）|
| 單答 / 複答 | **`marks` × 1 行** |
| Table | render 表格本身，blank cell 最低高度約 50pt 俾學生手寫 |
| Composite | 每 sub_question 按 sub_type 規則計，累加 |

行高 24-28pt，底線示意。

**Table render 要求**：
- 用 `<table>` element，`border-collapse: collapse; border: 1px solid #000;`
- Header row 灰底
- Blank cell（`____`）render 成空白單元格，最低高度 50pt
- 非 blank cell 直接顯示原文

### ② 題目順序固定
出卷時按呢個順序排：
1. 語譯（translate）
2. 三式判斷（judge）
3. 多項選擇（mc）
4. 多選題（multi_mc）
5. 單答題（short）
6. 複答題（long）
7. 表格題（table）
8. 複合題（composite）

每組標題：「一、語譯（共 X 分）」「二、判斷題（共 X 分）」⋯⋯（標題禁止 page-break-after）

### ③ 題庫檢索（題庫管理分頁）
頂部 filter bar，可多重組合：
- **關鍵字搜尋**（搜 text + answer）
- **範文 multi-select**（題目 source 係 array 時，array 中任一配中即計入）
- **來源 multi-select**
- **題型**（8 個 checkbox）
- **分數範圍** min-max
- **排序**（範文／題型／分數／加入時間，升降序）
- 「清除所有篩選」按鈕

即時更新，顯示「目前 X 條 / 總共 Y 條」。

### ④ PDF 預覽（出卷分頁）
抽完題後**先 render A4 預覽**（白底、有 padding、字體大小同實際列印一樣），預覽下方按鈕：
- 🖨️ **列印 / 儲存 PDF**（`window.print()`）
- 🔄 **重新抽題**（同條件再抽）
- ✏️ **手動換題**（點題目彈 modal 揀替換）

### ⑤ 種子碼（出卷分頁）
- 抽題前有「種子碼」input box（可留空 = 隨機）
- 留空抽題：**自動生成 8 位 alphanumeric 種子碼**，預覽上方清楚顯示（例：`種子：A7K2M9XQ`）
- 用戶複製貼返入去 + 完全相同篩選條件 → **抽出嚟一模一樣**
- 卷頭可選擇性印細字種子碼
- 用 seeded PRNG（mulberry32 + 種子 hash），**禁用 `Math.random()`**

### ⑥ 批量輸入
- 自動偵測 `[` / `{` 開頭 = JSON；否則用 `===` 文字格式
- 頂部 dropdown 揀／新增「呢批題目嘅來源」
- 預覽 → 確認加入
- 錯誤訊息具體：
  - 「第 3 條缺少『答案』」
  - 「第 5 條（單答題）必須填分數」
  - 「第 7 條 table 嘅 rows 欄數同 headers 唔一致」
  - 「第 9 條 composite 缺 sub_questions 或少於 2 個 sub」

## 額外要求
- 介面繁體中文，文案可帶廣東話口吻
- 偵測舊資料用「唐詩三首」「宋詞三首」做 source 時彈窗提示
- 備份格式 `{questions, sources, origins}`
- 出卷時，跨篇章題目（source 係 array）：用戶喺出卷揀範圍勾揀任何一篇範文都會抽到呢題；但同一張卷入面唔好重複出現

---

請幫我**由零開始寫**呢個單一 HTML 檔案，齊曬以上功能。
