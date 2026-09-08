# CLAUDE.md — costar-game（CO-STAR 卡牌配對練習遊戲）

依 CO-STAR 提示詞框架設計的拖放卡牌配對遊戲，玩法骨架**完整比照姊妹專案 `互動遊戲/crispe-game/`**（CRISPE 卡牌配對），只替換框架與題庫；CO-STAR 六欄位定義／中文標籤沿用 `行銷內容工具/ai-prompt-generator/` 既有的 `FRAMEWORKS.costar` 慣例（背景/目標/風格/語氣/受眾/輸出格式）。單檔前端，無建置步驟、無框架、零外部資源，直接開啟 `index.html`（`file://`）或以靜態伺服器託管即可。

## 與 crispe-game 的差異

- **6 格而非 5 格**：CO-STAR 的六個字母（C-O-S-T-A-R）恰好 1:1 對應六個類別（Context/Objective/Style/Tone/Audience/Response），不像 CRISPE 需要把 Capacity+Role 合併成一格「CR」——這次沒有壓縮，`SLOTS` 直接 6 筆。
- **每回合 5 主題 × 6 類別 = 30 張卡**（原版 25 張）。
- **計分公式改用比例換算**：6 格不能被 20 整除，`score = correct===6 ? 20 : Math.round(correct/6*20)`；6 格全對固定給整數 20，非滿分才四捨五入，5 題全對加總仍精準等於 100（維持與 CRISPE 版一致的「滿分 100」心理錨點）。
- **視覺主題改為「星空觀星桌」**：深藍夜色氈布（`--felt`系列改藍色調）＋石墨/鋼灰色頂欄（原版是綠色氈布＋原木），呼應 CO-STAR 的「STAR」意象；`body` 背景疊加數個小圓點 radial-gradient 模擬星點。LED 計時字改冰藍色（`--led:#8ec9f2`）與原版薄荷綠區隔。favicon 改用 `✨` emoji（原版 `🎴`）。
- **`THEME_COLORS` 換成「星系色」5 色**（星塵粉/極光綠/曙光橙/深空藍/銀河紫），與原版珊瑚紅/橘子橙/草地綠/湖水藍/葡萄紫區隔但用途相同（依抽題順序指派、logo tiles 循環對應 6 個字母，`'COSTAR'.split('')` 取代 `'CRISPE'.split('')`）。
- **CSS 網格欄數**：`.slot-row` 從 `repeat(5,1fr)` 改 `repeat(6,1fr)`，RWD 斷點沿用 900px（→3欄兩排）／620px（→2欄三排）。
- **題庫全新 20 組**，情境刻意與 crispe-game 的 20 組不重複（改用咖啡館、麵包店、花店、街舞教室、補習班、動物醫院、共享辦公室、腳踏車行、甜點店、攝影工作室、文物館、髮廊、駕訓班、搬家公司、清潔公司、桌遊店、小農市集、密室逃脫、接案設計師、健身房等情境），維持「虛構、台灣情境、卡牌文字不含關鍵字提示」原則。
- **localStorage key 全部改前綴**：`costarGameState`／`costarGameLeaderboard`／`costarGameMarquee`（與 crispe-game 的 `crispeGame*` 互不衝突，可同機同瀏覽器並存測試）。

## 玩法規則（沿用 crispe-game 骨架，僅代號改變）

- 每局從題庫隨機抽 5 個主題，每主題 6 張卡（共 30 張），依主題分 5 色。玩家把卡拖入正確提示格；放滿 6 格才能按「✔ 確認」判分；確認後格子亮綠（✓）／紅（✗）並自動彈出答案核對頁；「↻ 重新開始」＝重進當前主題。主題可任選、可重挑戰，分數以最近一次確認為準。
- 全域三鍵：「▶ 開始」／「⏸ 暫停」／「🏁 完成」，行為與 crispe-game 逐字相同。
- 五主題全數確認且總分 100 → 恭賀畫面＋彩帶＋完成音樂（`congratsShown` 旗標防重複觸發）。

## 架構（單一 index.html，程式邏輯與 crispe-game 共用同一套實作模式）

- **資料**：`QUESTION_BANK`（20 組，每組 `{id, title, desc, cards:{C,O,S,T,A,R}}`）、`SLOTS`（六格引導語）、`THEME_COLORS`（5 色，依抽出順序指派）。
- **狀態**：全域 `S = {drawn, results, totalMs, started, finished, congratsShown}`，存 localStorage（key: `costarGameState`）；重新整理後回「待命」狀態，按「開始」續走，總時間與各主題成績保留，進行中卡牌位置不持久化。
- **計時／拖曳／音效／彩帶**：與 crispe-game 逐字相同的實作（200ms tick 計時；Pointer Events 拖曳＋`forceCancelDrag()` 自我修復；WebAudio 合成音效 `sfx.place/good/bad/perfect/fanfare`；Canvas 手繪成績圖；`prefers-reduced-motion` 減敏處理），只有顏色常數與字數/分數相關文案不同，未重新設計機制本身。

## 上線（2026-09-08，已比照 crispe-game 模式但改用 Actions workflow）

已推公開 GitHub repo：<https://github.com/M255525/costar-game>，並用 **GitHub Pages Actions workflow**（非 legacy branch-source）部署上線：<https://m255525.github.io/costar-game/>。與 crispe-game 本身用的 legacy branch-source 設定不同——crispe-game 是較早期專案的既定設定未變更，本專案改採本工作區近期慣例（Actions workflow），對玩家體驗與網址完全無差異，純技術面選型。

**頂部跑馬燈**：抓取工作區共用的 Google Sheet 公告內容，做法逐字沿用 crispe-game（同一個 Apps Script 網址，空序號 POST，只取 `marquee` 欄位），`localStorage` key 改 `costarGameMarquee`；`body.has-marquee` / `.topbar` 的 26px 位移邏輯不變。

**使用警語＋創作者資訊**：文字與 crispe-game 逐字相同，放在 `#menuView` 的 `.menu-hint` 下方。

## Google Sites 嵌入版（platform/）

`platform/CO-STAR卡牌配對-GoogleSites嵌入用.html` 是供 Google 協作平台「插入 → 嵌入 → 嵌入程式碼」貼上的變體，做法比照 crispe-game：即 index.html 去掉 `<!DOCTYPE>`／`<html>`／`<head>`／`<body>` 外殼、只留 `<meta charset>`＋`<style>`＋內容＋`<script>` 的片段；同資料夾的 `一鍵複製-貼到GoogleSites.bat` 會把嵌入碼複製到剪貼簿（嵌入框建議拉高至少 900px）。**修改 index.html 後必須重新產生嵌入版**（於本專案根目錄執行）：

```bash
python -c "import re,io;src=io.open('index.html',encoding='utf-8').read();style=re.search(r'<style>.*?</style>',src,re.S).group(0);body=re.search(r'<body>\n(.*)\n</body>',src,re.S).group(1);io.open('platform/CO-STAR卡牌配對-GoogleSites嵌入用.html','w',encoding='utf-8').write('<meta charset=\"UTF-8\">\n'+style+'\n\n'+body+'\n')"
```

Sites 的沙箱 iframe 可能禁 localStorage——程式內 save/load 已包 try/catch，屆時只是不記分數、遊戲照玩。

## 單題答對彩帶（2026-09-08）

原本彩帶只在「5 主題全對、總分 100」的恭賀畫面／結算畫面觸發；使用者要求**每一題（單一主題）只要 6 格全對，當下就要灑花**，不必等到全部主題都完成。`btnConfirm` 判分邏輯的 `allCorrect` 分支已補上 `confetti()`（與 `sfx.perfect()` 同一個 `setTimeout(...,650)` 一起觸發），最終「5 主題全對」的彩帶（`showCongrats()`／`showResult()` 的 `total===100` 分支）維持不動、不受影響——兩者各自獨立觸發，單題與總結完成都會灑花。

## 指令

無建置／測試指令。修改後直接開瀏覽器驗證，或 `python -m http.server <port> --directory costar-game` 暫起伺服器測完關閉。自動化驗證以 Playwright `browser_evaluate` 派發 PointerEvent 模擬拖曳最可靠；關鍵流程放在單一 evaluate 內完成，避免真人操作同一個可見瀏覽器視窗干擾跨呼叫的狀態斷言。
