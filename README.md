# Word → CMS 文章排版轉換工具

把 Word 文章（.docx）轉成符合公司 CMS 樣式規則的 HTML，方便複製貼進老舊 CMS。
**純前端、無後端**，所有處理都在使用者的瀏覽器本機完成，**檔案不會上傳**到任何伺服器。

這是原本那段在 Google Colab 執行的 Python 轉換腳本的網頁版本：上傳 .docx → 套用樣式規則（H2/H3 樣式、自動目錄、`<p>&nbsp;</p>` 間距、表格框線等）→ 複製貼進 CMS。

---

## 給同事：怎麼用

1. **在 Word 或 Google Docs 把文章寫好**，並用「**標題樣式**」設定好標題層級：
   - H1 / H2 / H3 一定要用文件的「標題 1 / 標題 2 / 標題 3」樣式，**不能只是把字放大加粗**，否則工具偵測不到標題。
2. **存成 / 下載成 `.docx`**：
   - **Google Docs**：檔案 → 下載 → **Microsoft Word (.docx)**
   - **Word**：直接另存為 `.docx`
3. 開啟工具網址（見下方部署後的連結），把 `.docx` **拖進上傳區**或點選檔案。
4. 右邊會自動顯示套好樣式的**預覽**。
5. 按 **「複製 HTML 原始碼」**，到 CMS 切換到「**原始碼 / `<>`**」模式後 **貼上（Ctrl+V）**。
6. 萬一某 CMS 貼上有狀況，可改按 **「下載 .html」** 取得檔案。

> ⚠️ **文章目錄要能正常跳，一定要用「複製 HTML 原始碼」貼進 CMS 的原始碼模式。**
> 若改用「複製內容（所見即所得）」貼上，瀏覽器會把目錄的相對連結（`#toc-h2-0`）改寫成連回本工具網址，導致在 CMS 上點目錄跳不到對應段落。所見即所得只適合沒有目錄、或不在意目錄跳轉的情況。

### 重要行為說明
- **標題務必用「標題樣式」**（Heading 1/2/3）標記，這是自動產生「文章目錄」與標題樣式的依據。
- **H1 不會輸出**（比照原本的工具，文章大標通常由 CMS 文章標題另外設定）。
- H2 標題色碼維持原腳本的 `#07788`（原腳本就有的值，未更動）。
- **圖片不會輸出**（與舊工具一致）；圖片請在 CMS 內另外處理。
- 表格、粗體、斜體、超連結、項目／編號清單都會保留並套用樣式。

---

## 給維護者：部署到 GitHub Pages

1. 把整個資料夾（`index.html`、`vendor/`、`README.md`）push 到 GitHub repo。
2. 進 repo → **Settings** → **Pages**。
3. **Source** 選 `Deploy from a branch`，Branch 選 `main`、資料夾 `/ (root)`，按 **Save**。
4. 等一兩分鐘，會得到網址：`https://<你的帳號>.github.io/<repo 名稱>/`
5. 把這個網址分享給同事即可。

之後要改樣式或規則，直接編輯 `index.html` 再 push，GitHub Pages 會自動更新。

---

## 本機測試

直接用瀏覽器打開 `index.html` 即可（不需伺服器）。
若想模擬正式環境，可在資料夾內跑一個簡單靜態伺服器後開 `http://localhost:8000`。

---

## 技術說明（給維護者）

- `index.html`：UI + CSS + 轉換邏輯。
- `vendor/mammoth.browser.min.js`：[mammoth.js](https://github.com/mwilliamson/mammoth.js)，**直接讀 .docx 檔案內部結構**轉成乾淨語意 HTML（標題→`<h1>/<h2>/<h3>`、粗體→`<strong>`、表格→`<table>` 等）。內嵌進 repo，無需 CDN、可離線。
- 流程：`mammoth.convertToHtml(.docx)` → 乾淨 HTML → `convertDoc()` 套用 CMS 樣式規則（標題樣式、自動目錄 `applyAutoTocAndSmooth`、表格 `tableToHtml`、`<p>&nbsp;</p>` 間距）→ 預覽 / 複製 / 下載。
- **為什麼讀 .docx 而不是「複製貼上」**：先前版本嘗試讓使用者把內容貼進網頁編輯器再轉換，但 Word / Google Docs 的剪貼簿 HTML 雜亂且各來源不一致，會導致表格沒轉、標題跑版。改成直接讀 `.docx` 檔案本身（結構乾淨），就根除了這個問題。
- 複製採用 Clipboard API 寫入 `text/html`（rich paste）；不支援時自動退回 `execCommand('copy')`。
- 轉換規則（`convertDoc` / `applyAutoTocAndSmooth` / `tableToHtml`）是原 Python 腳本對應函式的 JavaScript 移植。
