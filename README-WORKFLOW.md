# 網站維護說明 — 黃仲誼老師 AICV Lab

給接手這個工作資料夾的任何 AI 助理／未來的自己看的說明文件，避免每次都要重新解釋。

## 基本資訊

- **正式網址**：https://schumihuang.github.io/
- **工作目錄**：`Z:\個人網站維護\schumihuang.github.io`（Windows 網路磁碟，git 倉庫）
- **GitHub 倉庫**：https://github.com/schumihuang/schumihuang.github.io
- **網站類型**：單頁式 HTML（`index.html` 一個檔案包辦所有內容與樣式），部署在 GitHub Pages，push 到 `main` 分支後自動建置上線
- **這個網路磁碟偶爾會有所有權問題**，如果 git 指令出現 "dubious ownership" 錯誤，執行：
  ```bash
  git config --global --add safe.directory "<顯示的路徑>"
  ```

## 檔案結構

| 檔案 | 用途 |
|---|---|
| `index.html` | 網站全部內容（HTML + inline CSS），唯一需要編輯的檔案 |
| `robots.txt` | 允許所有爬蟲，指向 sitemap |
| `sitemap.xml` | 單頁網站的 sitemap（只有首頁一個 URL） |
| `googlecdba73bf57f25c28.html` | Google Search Console 網站驗證檔，**不可刪除** |
| `assets/` | 圖片（大頭照 AT.png、CIHuang*.png 等） |
| `SQUARESPACE-REBUILD-GUIDE.md` | 舊的 Squarespace 重建指南，已不採用（改用 GitHub Pages 直接部署），保留僅供參考 |

## 標準工作流程（每次修改都照這個順序）

1. 用 `Read` / `Grep` 定位要改的內容
2. 用 `Edit` 修改 `index.html`
3. 提交並推送（**務必先 pull --rebase**，因為使用者有時會直接在 GitHub 網頁上傳檔案，導致遠端跟本地不同步）：
   ```bash
   cd "Z:/個人網站維護/schumihuang.github.io"
   git add index.html
   git commit -m "說明這次改了什麼"
   git pull --rebase origin main
   git push origin main
   ```
4. 等 GitHub Pages 建置完成再回報使用者，用 `gh api` 輪詢，不要用固定時間的 `sleep`：
   ```bash
   target="$(git rev-parse HEAD)"
   for i in $(seq 1 8); do
     latest="$(gh api "repos/schumihuang/schumihuang.github.io/pages/builds/latest" 2>/dev/null | grep -o '"commit":"[^"]*"' | cut -d'"' -f4)"
     status="$(gh api "repos/schumihuang/schumihuang.github.io/pages/builds/latest" 2>/dev/null | grep -o '"status":"[^"]*"' | cut -d'"' -f4)"
     if [ "$latest" = "$target" ] && [ "$status" = "built" ]; then echo "✅ built"; break; fi
     sleep 12
   done
   ```
5. 最後用 `curl` 或 `WebFetch` 直接查正式網址，確認內容真的有更新（本機 shell 的 curl 偶爾會遇到 TLS 連線中斷，若 `HTTP 000` 就改用 `WebFetch` 驗證，不要因此誤判失敗）

## 網站內容慣例（很重要，改內容前先看這段）

### Publications 區塊（`#publications`）
- 每篇論文結構：`authors` → `title` → `roles`（角色徽章，選填）→ `meta`（期刊/年份/卷期/DOI）→ 右側 `badge`（SCI / Q1 / Accepted / EI）
- **角色徽章**（金色 `.role-badge`）：只用「第一作者 · First Author」「通訊作者 · Corresponding Author」，兩者都是就疊加兩個徽章。**不要**寫「PI」「Co-PI」這種混合英文縮寫，使用者明確要求中文優先、英文完整拼出
- **期刊指標徽章**（藍色 `.role-badge.sci-badge`）：格式固定為 `SCI · IF X.X`，只有官方確認是 SCI 期刊才加；EI 期刊寫「EI」不寫 IF；**絕對不能自己編 IF 數字**，沒有可靠來源就只寫「SCI」不寫數字，並主動告知使用者還缺哪些
- **卷期/DOI 一律等正式出版才補**，Early Access（沒有 volume/issue，Crossref 顯示 `page: "1-1"`）就維持 `Accepted for publication`
- 查證 IF／卷期的方法：`https://api.crossref.org/works/<DOI>`（用 WebFetch 讀），比 WebSearch 準確很多

### News 區塊（`#news`）
- **固定只放最新 5 篇**，新增一篇就要把最舊的一篇移除
- 每則格式：`賀～論文獲期刊接受`（Accepted 但未正式出版）或 `賀～論文正式發表`（已有卷期/正式出版）+ 期刊名 + 角色（如有通訊作者）
- 新論文永遠加在 News 和 Publications 的**最上方**

### 專利（`#patents`）
- 中文用語是「**發明人**」不是「作者」；標籤統一「First Inventor · 第一發明人」
- 中英文標題**分開**列（不要合併成一行），維持原本 4 筆的獨立卡片

### 中英對照原則
- 大部分區塊採「英文為主 + 中文小字對照」，用 `.zh` 或 `.zh-p` class（`.zh-p` 會套用中文字體 `var(--tc)`）
- 職稱／機構名稱盡量中英並列（例：Assistant Professor 助理教授）
- 不要自己回譯或編造中文機構全稱，沒把握就直接問使用者

### 其他已知的設計決定
- Hero 大頭照目前用 `assets/AT.png`（曾經換過 CIHuang.png → CIHuang2 → 3 → 43 → AT，如果使用者又要換，記得檔案要 `git add` 一起推送，否則會 404）
- 手機版（`max-width:920px`）大頭照改為 `display:flex; order:-1` 置頂顯示，**不要**改回 `display:none`
- AICV Lab 旁邊原本有 4 個統計數字方塊，使用者認為沒意義已整組移除，**不要**自作主張加回去
- Project timeline（`#projects`）目前用 `style="display:none;"` **隱藏但保留內容**，使用者說之後視情況再顯示 —— 移除該行內聯樣式即可恢復
- 章節順序（由上而下）：About → Awards → Research → News → Publications → PI Experience → Patents → Projects(隱藏) → Lab → Service → Contact —— 使用者曾要求調整過順序，改動前先確認目前順序

## SEO / Google 索引相關

- `googlecdba73bf57f25c28.html` 是 Search Console 驗證檔，**不可刪除或修改**
- 新內容上線後如果使用者問「Google 還沒收錄」，先檢查：robots.txt 有沒有擋、有沒有 noindex meta tag、sitemap 能不能正常存取（`curl -I https://schumihuang.github.io/sitemap.xml`），通常都沒問題，純粹是 Google 爬取需要時間，建議使用者去 Search Console 手動「要求建立索引」

## 舊 Weebly 網站的參考資料

- 使用者原本的網站是 `schumi.weebly.com`（現已下線，404）
- 使用者提供過本機的 Weebly 匯出備份：`Z:\個人網站維護\Weebly\`，裡面 `Weebly_files/main_YEOJ.htm` 是**已發布網站的完整原始碼**（其他檔案多半只是編輯器介面的系統文字，沒有實質內容），內含完整中文計畫清單、計畫主持人經歷等珍貴原始資料 —— 如果使用者又問起「以前是不是有一份 XXX 資料」，先去這個檔案裡用 Grep 找關鍵字，很可能就在裡面
- 使用者也曾提供官方升等資料 docx（路徑在 G 槽，`教師升等` 資料夾），裡面有 JCR 官方排名與 IF 數字，比網路搜尋準確，如果使用者又提供類似文件，優先採用文件內數字，不要用網路查到的估計值
