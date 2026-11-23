# 🎱 Billiards Kids Academy — AGENTS & HANDOVER GUIDE

> **給接手 Agent (Remote Conversation) 的完整交接指南與進度手冊**  
> 本檔案記錄專案架構、開發歷程、已完成核心模組、代碼規範以及後續建議待辦事項（Next Steps）。

---

## 📌 專案概覽 (Project Overview)

* **專案名稱**：小小撞球神射手：四年級撞球科學與實戰學院 (Billiards Kids Academy)
* **當前版本**：`v4.2`（比照 Image 2 經典街機大球桌 1360px × 實木金屬包角 × 經典黃金 HUD × AI 智慧對戰旗艦版）
* **線上部署 (GitHub Pages)**：[https://zhijaysun0915.github.io/billiards-kids-academy/](https://zhijaysun0915.github.io/billiards-kids-academy/)
* **Git 遠端倉庫**：`https://github.com/zhijaysun0915/billiards-kids-academy.git` (分支：`main`)
* **本地工作目錄**：`/Users/jaysun/Downloads/billiards-kids-academy`
* **架構定位**：專為國小四年級學童與撞球初學者設計的教育型 Web 撞球模擬遊戲，深度整合 [運動島人 (Sport Islander) 撞球規則](https://sport-islander.com/billiards-rules/) 與真實力學科學。
* **技術棧**：原生 HTML5 Canvas 2D + 純原生 JavaScript (ES6+) + CSS3。**完全零外部相依套件 (Zero-dependency)**，所有音訊皆已轉碼為 Base64 內嵌，單檔即可離線完整運作。

---

## 📂 專案檔案結構 (Repository Structure)

```text
billiards-kids-academy/
├── AGENTS.md                   # 🤖 給接手 Agent 的交接文檔與開發手冊（本檔）
├── README.md                   # 📖 使用者面向的專案說明書與特色手冊 (v4.0)
├── index.html                  # 🚀 GitHub Pages 進入點 (核心遊戲單檔)
├── billiards_kids_academy.html # 🔄 與 index.html 100% 嚴格同步的備份副本檔
└── sounds/                     # 🎵 原始真實撞球錄音音效檔 (已 Base64 內嵌於 html 內)
    ├── ball_hit_sharp.mp3
    ├── ball_hit_soft.mp3
    ├── cushion_bounce.mp3
    ├── pocket_drop.mp3
    └── cue_strike.mp3
```

> [!IMPORTANT]
> **雙檔同步守則**：若修改了 `index.html`，**必須同步覆蓋至 `billiards_kids_academy.html`**（`cp index.html billiards_kids_academy.html`），確保兩者內容保持 100% 一致。

---

## 🏗️ 核心架構與物理引擎實作

### 1. 球桌幾何與 6 個經典球袋口 (`TABLE` & `POCKETS`，比照 Image 2 經典規格)
* **球台規格**：Canvas 解析度 `920 x 500`，有效球呢面積為 `800 x 400`（坐標範圍 `left: 60, top: 50, right: 860, bottom: 450`），母球與子球半徑 `ballRadius: 13.5`。
* **外觀視覺細節（比照 Image 2 經典規格）**：
  * **胡桃木實木邊框**：具備橫縱向木質纖維肌理、緞面光澤漸層與內緣立體 3D 陰影凹槽。
  * **3D 鍍鉻拋光金屬轉角包角 (Chrome Corner Castings)**：4 個邊角以弧形鍍鉻金屬圓角包覆，帶有高光金屬鏡面反射與固定鉚釘。
  * **中袋金屬修邊 (Side Pocket Brackets)**：木框於中袋兩側具備金屬壓板。
  * **中袋切齊桌布邊不向內凸出**：上中袋與下中袋的皮革座與深黑袋孔完整內嵌於長顆星木框中，使用 `ctx.clip()` 裁切於呢布水平邊界線（$y = \text{TABLE.top}$ 與 $y = \text{TABLE.bottom}$），保證袋口平直切齊桌布邊緣，絕不向球呢中央凸出！橡膠庫邊以 104° 夾角自然斜切入袋。
  * **錦標賽經典墨綠色球呢 (Emerald Green Felt)**：中央柔和聚光投射，並繪製發球開球線 (Kitchen line) 與英美式經典 "D" 形發球弧。
  * **18 顆常駐珍珠/黃金鑲嵌菱形點 (Diamond Sights)**：邊框均勻排列瞄準記號點。
* **6 塊獨立立體顆星橡膠條 (Cushion Rails)**：袋口兩側具備斜切導入口（Mouth Jaws）與高光接觸稜線，物理判定自然平滑導入，絕不卡球或產生隱形阻擋。

### 2. CCD 連續碰撞偵測與真實能量衰減 (避免球體無限回彈)
* **解析級接觸時間解算器 (Analytical Time-of-Impact Solver)**：
  * 在子步循環 (`subSteps = 8`) 中，求解一元二次方程：
    $$\|\mathbf{p}_2(t) - \mathbf{p}_1(t)\|^2 = (2R)^2 \quad (0 \le t \le 1)$$
  * 接觸瞬間立刻推進至臨界接觸點，嚴格保證碰撞法線角度精度達 $10^{-14}$，子球出射角度完全不偏角。
* **物理能量衰減與防無休止回彈模型**：
  * **顆星橡膠反彈係數**：由過往過度彈跳的 $0.90$ 修正為符合真實橡膠物性的 $0.66$，反彈 1~2 次後動能大幅衰減。
  * **球呢布雙重滾動摩擦力**：採用指數衰減（$0.982$ 每幀）結合庫侖動摩擦線性減速（$0.024\text{ px/frame}$），擊球後球體滾動沉穩紮實。
  * **球球碰撞彈性係數**：修正為 $0.92$（真實酚醛樹脂球物性），避免球堆開球時出現彈珠台式永動混亂。
  * **停球判定速度**：提升至 $0.22\text{ px/frame}$，徹底杜絕無意義的微速蠕動與延遲。

### 3. Image 2 經典滿版全景大球桌佈局與街機底欄 HUD (Arcade Mode)
* **全景滿版球桌 (Full Bleed Arcade Table)**：
  * 進入 8 號球或 9 號球比賽對戰時，自動切換至 `.match-mode-expanded` 與 `.arcade-fullscreen` 模式（亦可於頂端導覽列隨時點擊 `🖥️ 滿版全景大球桌 (Image 2 經典模式)` 手動切換）。
  * 隱藏右側側邊欄與上方教練卡片，球桌容器寬度擴展至 `1360px`，桌布寬度達 `98vw`，完全充滿視野！
* **Image 2 經典街機底欄 HUD**：
  * **左側進球收納槽 (POCKETED TRAY)**：即時陳列已進袋子球的微型立體圖示與花色球號。
  * **四大黃金立體雕刻數值儀表**：
    1. **MULTIPLIER**（當前賽制/模式：`8-BALL` / `9-BALL` / `STAGE`）
    2. **TIME / TURN**（出桿方與動態高光：`🔵 P1` / `🔴 P2`）
    3. **SCORE**（即時比分與進球統計）
    4. **RACK**（當前局數 `RACK 1`, `RACK 2`）
  * **核心操作四要素整合於下方**：
    1. **選桿切換**（🎱 標準打桿、🦘 專業短跳桿、🌀 扎桿弧線、✋ 自由擺球）
    2. **出桿力度**（💪 出桿力道 Power 滑桿與黃金刻度發光進度）
    3. **角度微調**（角度數值顯示與微調按鈕：$⏮ -2°, ↺ -0.5°, ↶ -0.1°, ↷ +0.1°, ↻ +0.5°, ⏭ +2°$）
    4. **出桿擊球大按鈕**（🚀 出桿擊球！點擊或空白鍵 Space 發射）

### 3. 🦘 3D 跳球物理與空中跨越 (Jump Shot Physics)
* **國際 WPA 規則合規力學**：
  * 嚴格杜絕違規鏟球（Scoop Foul）。真實跳球是利用短跳桿以高俯角（`35° / 45° / 55°`）由上往下重壓石板，由石板反作用力產生彈跳：
    $$v_{xy} = v \cdot \cos(\theta) \times 0.90, \quad v_z = v \cdot \sin(\theta) \times 0.65, \quad g = 0.60$$
* **空中跨越障礙球判定**：
  * 在 CCD 與球體碰撞檢測中，若飛行球體高度 $z > \text{ballRadius} \times 1.25$，直接忽略與地面障礙球的碰撞，達成真實跨越障礙球。
* **3D 視覺與動態地面陰影**：
  * 飛行時球體 Y 軸視覺偏移：`drawY = b.y - b.z * 0.85`，視差縮放：`scale = 1 + min(b.z/180, 0.35)`。
  * 呢布上即時投射隨高度淡化縮小的柔和橢圓陰影，並有頂部高度標籤（`🦘 98px`）。
* **拋物線導引與預測線**：
  * 即時計算拋物線軌跡、最高空頂點（`🦘 最高騰空`）、精準落地光圈（`🎯 落地點`），並檢測前方障礙球飛行高度（`✅ 越過 / ⚠️ 太低將撞到`）。

### 4. 🌀 扎桿弧線球 (Massé Shot)
* 立起球桿至 75 度重擊白球側翼，給予強烈側旋（English），利用陀螺儀進動（Gyroscopic Precession）與呢布微觀摩擦力，在前進時產生持續側向加速度：
  $$v_x \gets v_x - \sin(\theta) \cdot \kappa, \quad v_y \gets v_y + \cos(\theta) \cdot \kappa \quad (\kappa \text{ 隨時間衰減})$$
  使白球在球桌上劃出優美的 C 型弧線繞道進球。

### 5. ✋ 自由球直覺擺放 (Ball-in-Hand UX)
* 不必再艱難地在觸控螢幕上拖曳小半徑母球！
* 在自由球狀態下，**在球桌任意處輕點一下即可直接擺球**。
* 移動過程中即時顯示綠色（合法）/ 紅色（與其他球重疊）準心與光圈，並提供浮動捷徑列（`[開球線]`、`[球桌中央]`、`[✅ 確認擺放]`）。

### 6. 🔊 實體錄音音效系統與音量控制 (Authentic Audio Engine)
* 取代合成音，全面採用高品質實體酚醛樹脂撞球碰撞錄音、顆星沉悶鈍響、進袋落袋聲與皮頭出桿摩擦音。
* 內嵌 Base64 字串，具備音量動態調節、主音量滑桿（`0%~100%`）、一鍵靜音切換（`🔊 / 🔇`）與過關/獲勝和弦號角音效（`playVictoryFanfareSound`）。

### 7. 🤖 智慧 AI 對手決策引擎 (`computeBestAIShot` & `executeAIShot`)
* 支援 9 號球與 8 號球對戰模式切換 `👥 雙人2P`、`🐣 AI幼幼班`（附帶輕微角度誤差）、`🤖 AI神射手`（零誤差 + 高階解球）。
* **多層次決策樹**：
  1. **全台自由球智慧擺放**：當 AI 獲自由球時，自動在合法目標球與袋口連線後方尋找無遮擋最佳母球擺放點。
  2. **直線假想球進袋評估**：掃描所有合法目標球與 6 個袋口，檢查母球至假想球、子球至袋口路徑是否淨空，並評估切球角度 ($\cos\theta$) 與洗袋風險。
  3. **一顆星鏡面反射解球 (1-Cushion Kick Shot)**：當直線被障礙球封死時，計算上下左右四條顆星的鏡像反射點進行一顆星解球。
  4. **專業短跳桿飛越解球 (Pro Jump Shot)**：`AI神射手` 模式下，若與目標球之間僅隔一顆中距離障礙球，自動切換 `jump` 短跳桿凌空躍過障礙！

### 8. 🛠️ 自訂排球沙盒與分享碼系統 (`setupSandbox`, `exportSandboxRack`, `importSandboxRack`)
* 支援選取 `0~15` 號球點擊球桌擺放、直接按住任何球拖曳移動、點擊移除子球、4 種快速排陣預設集，以及 `BKA4:...` 關卡代碼匯出與匯入。

---

## ✅ 已完成功能清單 (Completed Features)

- [x] **手機與平板專屬響應式排版**
  - [x] 球桌懸浮出桿發射鈕（右下角即時拇指擊球）
  - [x] 手機直立/橫向常駐 Dock 底部控制列（角度微調、出桿按鈕）
  - [x] 手指觸控流暢旋轉瞄準與力道控制
  - [x] 水平滑動模式導覽列與關卡膠囊列
- [x] **桌台與球袋視覺重構 + 球桌外觀與粒子特效 (v3.1 + v4.0)**
  - [x] 6 個經典深邃黑色球袋口（中袋平直切齊桌布邊不向內凸出，4 角袋圓形）
  - [x] 6 塊立體雙層漸層顆星橡膠條（45 度斜切導入口與高光接觸稜線）
  - [x] 銀色金屬包角（Chrome Corner Castings）與皮革中袋座
  - [x] 呢布鑽石刻度（Diamonds 0~8）
  - [x] 4 款球桌呢布顏色主題切換（經典綠布、賽事藍布、英倫酒紅、皇家紫金）
  - [x] 進袋星光粒子、震波光圈、浮動文字與過關/獲勝滿天彩帶特效
- [x] **物理與控制引擎升級**
  - [x] CCD 連續碰撞檢測（解析級 TOI 解算，零偏角）
  - [x] 真實物理能量衰減與防無休止回彈（顆星反彈 0.68、雙重滾動摩擦阻力、球球碰撞 0.92）
  - [x] 8號/9號對戰專用放大球桌（1180px 寬，隱藏教具側邊欄，僅留選桿、力道、角度微調、出桿）
  - [x] 自由出桿無鎖定機制（移除灰色 disabled）
  - [x] 專業短跳桿（35° / 45° / 55°）與 3D 騰空跳球物理
  - [x] 跳球拋物線預測、頂點標記、落地點準心、障礙球高度檢測
  - [x] 扎桿弧線球（Massé）物理與弧線軌跡預測
  - [x] 自由球（Ball-in-Hand）全桌點擊任意處擺球與即時重疊檢測
- [x] **音效系統與音量控制 (v4.0)**
  - [x] 實體球體撞擊音、顆星反彈音、進袋音、皮頭出桿音
  - [x] 100% 離線 Base64 內嵌與跨平台 AudioContext 自動喚醒
  - [x] 音量滑桿（0%~100%）、一鍵靜音按鈕與勝利號角和弦音效
- [x] **遊戲與教學模式**
  - [x] 🎓 **10 關闖關特訓營 (v4.0 擴充)**：
    - 第 1 關：直球進袋與假想球瞄準
    - 第 2 關：切球角度大考驗
    - 第 3 關：推桿 vs 拉桿魔法
    - 第 4 關：一顆星鏡面反射解球
    - 第 5 關：下塞（左/右塞）顆星大轉彎
    - 第 6 關：做球走位雙球清台 (1號 ➔ 9號)
    - 第 7 關：🦘 神奇跳球 (Jump Shot) 飛越障礙球
    - 第 8 關：🌀 扎桿弧線球 (Massé Shot) 神奇大繞道
    - 第 9 關：⚡ 黃金傳球 (Combo Shot) 借力打力連鎖進袋
    - 第 10 關：👑 終極大魔王：三球走位一桿清台殘局挑戰
    - 全關卡支援「👀 看教練示範一桿」自動演示與 ⭐ 10 顆星徽章紀錄
  - [x] ⚔️ **雙人實戰對決 & 🤖 單人 PVE AI 對戰模式 (v4.0)**：
    - 支援 `👥 雙人2P`、`🐣 AI幼幼班`、`🤖 AI神射手` 即時切換
    - 9 號球對決（菱形排球，最小號碼優先規則，犯規全台自由球）
    - 8 號球對決（15 顆三角排球，大花小花球組自動判定，誤進 8 號球判負）
    - 上方專屬即時計分板與輪流出桿提示
  - [x] 🛠️ **自訂排球沙盒模式 (Sandbox Custom Rack, v4.0)**：
    - 0~15 號球自由點選擺放、球桌直接拖曳、點擊移除子球
    - 4 種預設排陣與 `BKA4:...` 關卡分享碼匯出/匯入
  - [x] 🧪 **自由實驗室**：
    - 下塞與桿法實驗室
    - 顆星鑽石實驗室
  - [x] 📖 **小小撞球規則與科學百科全書**：
    - 5 大章節（顆星反射、假想球、桿法下塞、8/9號球比賽規則、跳球與扎桿科學原理）

---

## 🚀 待辦事項與未來優化建議 (Next Steps / Backlog)

前期規劃之三大優先級待辦項目已於 `v4.0` 全部實作完成：
- [x] **優先級 1：單人電腦對戰 (PVE AI Mode)** — 已完成（支援幼幼班與神射手雙難度、自由球擺位、一顆星解球與跳球解障礙）
- [x] **優先級 2：球桌外觀自訂與視覺特效 (Cosmetics & Polish)** — 已完成（4 款球桌呢布配色、進袋粒子與慶祝彩帶、音量拉桿與靜音按鈕）
- [x] **優先級 3：花式解球與自訂排球沙盒 (Trick Shots & Sandbox Custom Rack)** — 已完成（擴充至 10 關特訓營、新增自訂排球沙盒與分享碼匯出/匯入）

---

## 🛠️ 開發與驗證指引 (Developer Commands)

### 1. 檢驗 JavaScript 語法正確性
在提交任何修改前，請務必執行 Node.js 語法檢驗腳本，防止未閉合標籤或重複宣告：
```bash
node -e '
const fs = require("fs");
const html = fs.readFileSync("index.html", "utf8");
const scriptMatches = [...html.matchAll(/<script[\s\S]*?>([\s\S]*?)<\/script>/gi)];
for (let i = 0; i < scriptMatches.length; i++) {
  new Function(scriptMatches[i][1]);
}
console.log("Syntax check passed!");
'
```

### 2. 同步雙檔案
```bash
cp index.html billiards_kids_academy.html
diff -u index.html billiards_kids_academy.html # 應無任何輸出
```

### 3. Git 提交與推播
```bash
git add -A
git commit -m "feat/fix: <說明描述>"
git push origin main
```
GitHub Pages 會自動觸發部署（通常約 1~2 分鐘內於 `https://zhijaysun0915.github.io/billiards-kids-academy/` 生效）。
