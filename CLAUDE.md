# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概況

單一檔案網站 `gta6-settings-guide.html`（「GTA 6 開局攻略集」）：把 Threads 貼文與 YouTube 影片整理成可勾選進度的攻略。沒有 build、lint、測試、套件管理；專案根目錄不是 git repo（git 倉庫在 `deploy/gta6-guide/`，見「發布到 GitHub Pages」）。使用者偏好繁體中文（台灣用語）、先給結論，改程式後要附回歸測試項目。

## 執行與驗證

- 本機預覽：`.claude/launch.json` 的 `gta6-guide`（`python -m http.server 8765 --bind 127.0.0.1`），開 `http://localhost:8765/gta6-settings-guide.html`。不要用 `file://`：相對路徑圖片會載不進預覽。
- 驗證清單：導覽各項計數（設定 0/80、金色武器 0/17、第一章 0/17、主線 0/94，總進度 0 / 208）；全部圖片載入（先把 `img.loading` 設 `eager` 再檢查 `naturalWidth`）；console 無錯誤；用 `resize_window` 模擬 800px 與 390px，確認 `scrollWidth <= innerWidth`（預覽窗格隱藏時 `innerWidth` 會是 0，會誤報溢出）。
- 預覽時 CSS 有 `scroll-behavior:smooth`，用 `window.scrollTo` 檢查位置前先設 `documentElement.style.scrollBehavior='auto'`。

## 架構（單檔內的資料驅動渲染）

HTML 只有各章節的空容器與說明文字，內容由檔尾 `<script>` 內的陣列渲染：

- `DATA`：設定篇四個分類（`#tabs`，每項可勾選）。
- `WEAPONS`（17 張卡，`#weaponList`）：`{id,name,sub,meta,steps:[[標題,說明,影片id,秒]],note,shots,extra}`。截圖用 `STEP_IMG[武器id]={步驟序號(0起算):[檔名或"map:地圖key",說明]}` 內嵌在對應步驟；沒配到步驟的 `shots` 由 `leftover()` 放在「其他畫面」。檔名含 `/` 視為完整路徑，否則走 `F(name)` → `assets/frames/<name>.jpg`。`MAPS` + `mapHtml()` 以 CSS 裁切整頁地圖截圖。
- `CH1`（`#ch1List`，17 項，可勾「已完成」）、`WR`（`#wrList`，摔角手）、`CARS`（`#carList`，車輛與賺錢，YG tech）：`{t,sec,d,q:[[秒,字幕原句]],warn,imgs}`；`WR`/`CARS` 由 `renderGroups(hostId,groups,videoId)` 渲染（無勾選），`CH1` 另有自己的渲染以支援勾選。`▶ 影片` 的秒數要用「實際看到畫面的秒數」，YouTube 描述欄章節時間有時對不上。
- 章節順序（建議觀看順序，DOM 順序須與導覽陣列一致）：設定、`#faq`（不在導覽）、`#tips` 開局十件事、`#moves`、`#community`、`#bychapter` 依章節查、`#wrestle`、`#chapter1`、`#missions`、`#heist` 搶劫教學、`#gtabase`、`#events`、`#weapons`、`#hidden`、`#cards`、`#cars`、`#money`。
- 章節標註：只寫來源明確寫出的章節（YouTube 標題、說明欄、字幕、貼文）。Somi 醬標題「第一章必拿 15 把」、見塵標題「第一章必拿 10 把」→ 武器卡 `meta` 的「章節」欄；梅卡梅、阿翊標題沒寫章節。`#bychapter` 是索引，來源沒寫的歸「來源沒寫章節」，不猜。新增含章節資訊的內容時，要同步更新 `#bychapter` 與武器卡章節欄。
- Facebook Reel／社團貼文：登入牆會把畫面蓋成淡灰，字幕很淡；播放到後半可能出現播放錯誤。讀不清楚就只寫標題與讀得到的片段，並明說。導覽有分組標題（陣列中 id 為 `"#"` 的項目，`.ngrp`，≤520px 隱藏）。
- 去重原則：同一件事只在一處寫完整，其他章用連結指過去（`<a href="#cars">`）。登記、追蹤器、出售價值等名詞放在 `#cars` 開頭的「名詞說明」；通用的取樣、簡轉繁、未驗證標籤說明放在頁首「怎麼讀這份攻略」，各章只留取樣間隔。新增內容前先搜尋是否已有同主題。
- 社群內容拆成三章，皆純靜態 HTML、每則是 `<details class="chap">`（`<span class="no cnum">` 編號要連號）：`#community` 社群小技巧與規則（2 則）、`#events` 隨機事件與彩蛋（4 則，飛機失事已合併）、`#money` 社群賺錢與載具（3 則：倉庫刷錢原版與優化版、免費拿車 4 種做法、銀行金庫；補充 `#cars`）。用 Python 依標題字串切區塊時，標題可能也出現在別則的內文裡（會切錯區塊），先確認標題唯一。第 1、2 則免費痛車的圖在 `assets/posts/`（Threads 只抓得到縮圖，約 210 到 320px），Reel 畫面在 `assets/frames/ez-*`、`zico-*`。`#gtabase` 是 GTABase 互動地圖用法（截圖 `assets/tips/gtabase-map.jpg`）。重排章節可用 Python 以 regex 切 `<section class="tab" id="…">` 區塊再依序拼回，並同步改導覽陣列。
- 讀不到的來源：IG Reel `DgHz6Wfouva` 標示年齡限制、需登入，沒有整理（不要為了讀而登入）。
- IG／FB Reel 讀法：隱藏登入 `[role=dialog]`，直接對 `video` 做 `__seek`；FB 描述的「查看更多」可用 JS 點開取全文。
- Threads 貼文讀法：`get_page_text` 取文字與留言；圖片 URL 要連同原本的查詢字串一起下載（改參數會 403），用 `curl.exe` 存檔；登入視窗不要登入，可用 CSS 隱藏 `[role=dialog]`。
- `MISSIONS`：5 章 94 項任務。
- 進度存在 `localStorage` 的 `gta6-guide-applied-v1`（id → 1）；`allIds` 收集所有可勾選項供計數。**更改 id 會讓舊勾選失效**（武器 id 已換過一次）。
- 導覽列由 JS 依固定順序建立（`[["weapons",…],…]` 陣列，在 `MISSIONS` 渲染之後），新增章節要同步改：HTML `<section id>`、該導覽陣列、頁尾來源清單、首頁 hero 說明文字。
- 圖片放大用單一 `<dialog id="lb">`，任何 `.zoom[data-full]` 按鈕都會觸發。
- 版面：寬度 > 520px 時導覽固定在左側（`.shell` 用 `clamp()` 欄寬），≤ 520px 才改成頂部單列橫向捲動。不要把斷點調回 860px（使用者曾回報導覽被移到上方）。

## 發布到 GitHub Pages

- 線上版：倉庫 `https://github.com/eric112310123/gta6-guide`（分支 `main`），網址 `https://eric112310123.github.io/gta6-guide/`。Pages 從 `main` 分支部署，每次 push 後約 1 到 2 分鐘自動更新（Actions 分頁的 `pages build and deployment` 綠勾）。網址是公開的；使用者所屬團隊曾禁止分享給他人，發布與否由使用者決定，推送前要先問。
- git 倉庫就是 `deploy/gta6-guide/`（含 `.git`），內容是 `index.html`（= 根目錄 `gta6-settings-guide.html` 的複本）加 `assets/`。`deploy/gta6-guide.zip` 是同內容的離線包（用 .NET `ZipArchive` 建，路徑用正斜線）。
- **重新產生 deploy 時絕對不能刪整個資料夾**：`.git` 是隱藏資料夾，`Remove-Item` 會先刪掉它（曾因此弄丟，靠重新 clone 遠端補回）。只做覆蓋：`Copy-Item` 把 `gta6-settings-guide.html` 複製成 `index.html`、`assets\` 同步過去（新增檔案要補上），再重建 zip。
- 發布流程：(1) 同步檔案到 `deploy/gta6-guide/`；(2) `git status` 確認只有預期的變更；(3) `git add -A`、`git commit -m "…"`、`git push`（需使用者同意）；(4) 用 `curl.exe -s -L https://eric112310123.github.io/gta6-guide/?v=<隨機數>` 檢查新章節的 id 是否出現，並對新圖 URL 取得 200。內建瀏覽器可能擋掉 github.io，用 curl 驗證即可。
- 若 `.git` 又不見：`git clone https://github.com/eric112310123/gta6-guide.git <暫存資料夾>`，把裡面的 `.git` 複製回 `deploy/gta6-guide/`，`git status` 就會顯示本機相對遠端的差異。
- 瀏覽器快取：看不到新版時 Ctrl+F5，或網址加 `?v=N`。新增圖片一定要一起 commit，否則線上缺圖。

## 內容與資料流程

- 圖片在 `assets/`：`frames/`（影片截圖 JPG，命名前綴 somi-、meka-、jc-、yg-、ch1-、c1-、wr-）、`maps/`、`sniper/`、`cards/`、`tips/`。影片截圖版權屬原作者，頁尾已註明。
- 讀 YouTube 影片的做法：在內建瀏覽器開影片，等廣告播完（不跳過），暫停 `video`，注入 `window.__seek(t)`（跳到 t、播 0.7 到 0.9 秒、暫停），再用 `computer zoom` 截圖播放器區塊。戲院模式播放器約 640×360；把 `#movie_player` 與 `video` 用 CSS 固定成 1024×576 並隱藏其他元素，可取得 800×450 的清楚畫面。描述欄章節可由 `window.ytInitialPlayerResponse.videoDetails.shortDescription` 取得；字幕 API 取不到，只能讀畫面燒錄字幕。
- 截圖以 PowerShell + `System.Drawing` 把 PNG 轉 JPG（品質 82 到 84）存入 `assets/frames/`。檔名對應只依取樣秒數時，存檔後用 Read 看一眼確認內容，不要憑猜測命名。
- 內容原則：只寫讀得到的字幕與畫面；沒取樣到的寫「影片沒取樣到」；留言說法標「留言說法・未驗證」；不編造座標（來源沒有座標，改用地圖截圖）。各影片衝突（如 KZ-59 位置）並列並請使用者以實際為準。字幕 `q` 要附秒數，`▶ 影片` 連結用 `https://youtu.be/<id>?t=<秒>`。
- 來源影片 id：Somi 醬 `lvZveGgR0tQ`、玫玫物語 `-xLXo_dnax4`（第一章防漏）、梅卡梅 `tXsAMlrkDTo`、見塵 `ACiUCjoH91o`、阿翊 `HbwzIGnaU7c` / `bk07Th-bXSk`、YG tech `472k7Y0ttoA`（車輛與賺錢，仍只有章節標題）、摔角手 `3ko7ycLUULk`。
