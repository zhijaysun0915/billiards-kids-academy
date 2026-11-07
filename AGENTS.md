# 🎱 Billiards Kids Academy — AGENTS & HANDOVER GUIDE

> **給接手 Agent (Remote Conversation) 的完整交接指南與進度手冊**  
> 本檔案記錄專案架構、開發歷程、已完成核心模組、代碼規範以及後續建議待辦事項（Next Steps）。

---

## 📌 專案概覽 (Project Overview)

* **專案名稱**：小小撞球神射手：四年級撞球科學與實戰學院 (Billiards Kids Academy)
* **當前版本**：`v3.0`（專業短跳桿 Jump Shot × 自由球直覺點擊 × 扎桿弧線 Massé × 8 關特訓營全能版）
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
├── README.md                   # 📖 使用者面向的專案說明書與特色手冊 (v3.0)
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

### 1. 球桌幾何與 6 個經典圓形球袋口 (`TABLE` & `POCKETS`)
* **球台規格**：Canvas 解析度 `920 x 500`，有效球呢面積為 `800 x 400`（坐標範圍 `left: 60, top: 50, right: 860, bottom: 450`），母球與子球半徑 `ballRadius: 13.5`。
* **6 個圓形球袋口 (`POCKETS`)**：
  * 徹底捨棄舊式方形或切角開口，依照經典美式球台全面重構為**圓形深黑球袋口**。
  * 包含 4 個角袋（銀色金屬轉角包角 Chrome Corner Castings）與 2 個腰袋（上中袋、下中袋）。
  * 6 塊獨立立體顆星橡膠條（Cushion Rails），袋口兩側具備 45 度斜切導入口（Mouth Jaws）與高光接觸稜線，物理判定自然平滑導入，絕不卡球或產生隱形阻擋。

### 2. CCD 連續碰撞偵測 (Continuous Collision Detection)
* **解析級接觸時間解算器 (Analytical Time-of-Impact Solver)**：
  * 在子步循環 (`subSteps = 8`) 中，求解一元二次方程：
    $$\|\mathbf{p}_2(t) - \mathbf{p}_1(t)\|^2 = (2R)^2 \quad (0 \le t \le 1)$$
  * 接觸瞬間立刻推進至臨界接觸點，嚴格保證碰撞法線角度精度達 $10^{-14}$。
  * 徹底解決傳統離散步長「穿透後再排斥」造成的出射角度微偏角，使子球實際行進路線與「黃色假想球預測線」100% 完全重合。

### 3. 🦘 3D 跳球物理與空中跨越 (Jump Shot Physics)
* **國際 WPA 規則合規力學**：
  * 嚴格杜絕違規鏟球（Scoop Foul）。真實跳球是利用短跳桿以高俯角（`35° / 45° / 55°`）由上往下重壓石板，由石板反作用力產生彈跳：
    $$v_{xy} = v \cdot \cos(\theta) \times 0.92, \quad v_z = v \cdot \sin(\theta) \times 0.68, \quad g = 0.60$$
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

### 6. 🔊 實體錄音音效系統 (Authentic Audio Engine)
* 取代合成音，全面採用高品質實體酚醛樹脂撞球碰撞錄音、顆星沉悶鈍響、進袋落袋聲與皮頭出桿摩擦音。
* 內嵌 Base64 字串，具備音量動態調節與音高隨機微偏移（Pitch Jitter），跨平台（iOS Safari / Android Chrome / 桌面）手勢自動解鎖。

### 7. ⚡ 零延遲自由出桿控制
* 徹底解除擊球按鈕灰色鎖定限制，出桿按鈕永不反灰，隨時可按；即使前一桿未完全停下亦能隨時發射下一球。

---

## ✅ 已完成功能清單 (Completed Features)

- [x] **手機與平板專屬響應式排版**
  - [x] 球桌懸浮出桿發射鈕（右下角即時拇指擊球）
  - [x] 手機直立/橫向常駐 Dock 底部控制列（角度微調、出桿按鈕）
  - [x] 手指觸控流暢旋轉瞄準與力道控制
  - [x] 水平滑動模式導覽列與關卡膠囊列
- [x] **桌台與球袋視覺重構**
  - [x] 6 個經典深邃黑色圓形球袋口（4 角袋 + 2 腰袋）
  - [x] 6 塊立體雙層漸層顆星橡膠條（45 度斜切導入口與高光接觸稜線）
  - [x] 銀色金屬包角（Chrome Corner Castings）與皮革中袋座
  - [x] 呢布鑽石刻度（Diamonds 0~8）
- [x] **物理與控制引擎升級**
  - [x] CCD 連續碰撞檢測（解析級 TOI 解算，零偏角）
  - [x] 自由出桿無鎖定機制（移除灰色 disabled）
  - [x] 專業短跳桿（35° / 45° / 55°）與 3D 騰空跳球物理
  - [x] 跳球拋物線預測、頂點標記、落地點準心、障礙球高度檢測
  - [x] 扎桿弧線球（Massé）物理與弧線軌跡預測
  - [x] 自由球（Ball-in-Hand）全桌點擊任意處擺球與即時重疊檢測
- [x] **音效系統**
  - [x] 實體球體撞擊音、顆星反彈音、進袋音、皮頭出桿音
  - [x] 100% 離線 Base64 內嵌與跨平台 AudioContext 自動喚醒
- [x] **遊戲與教學模式**
  - [x] 🎓 **8 關闖關特訓營**：
    - 第 1 關：直球進袋與假想球瞄準
    - 第 2 關：切球角度大考驗
    - 第 3 關：推桿 vs 拉桿魔法
    - 第 4 關：一顆星鏡面反射解球
    - 第 5 關：下塞（左/右塞）顆星大轉彎
    - 第 6 關：做球走位雙球清台 (1號 ➔ 9號)
    - 第 7 關：🦘 神奇跳球 (Jump Shot) 飛越障礙球
    - 第 8 關：🌀 扎桿弧線球 (Massé Shot) 神奇大繞道
    - 全關卡支援「👀 看教練示範一桿」自動演示與 ⭐ 8 顆星徽章紀錄
  - [x] ⚔️ **雙人實戰對決模式**：
    - 雙人 9 號球（菱形排球，最小號碼優先規則，犯規全台自由球）
    - 雙人 8 號球（15 顆三角排球，大花小花球組自動判定，誤進 8 號球判負）
    - 上方專屬即時計分板與輪流出桿提示
  - [x] 🧪 **自由實驗室**：
    - 下塞與桿法實驗室
    - 顆星鑽石實驗室
  - [x] 📖 **小小撞球規則與科學百科全書**：
    - 5 大章節（顆星反射、假想球、桿法下塞、8/9號球比賽規則、跳球與扎桿科學原理）

---

## 🚀 待辦事項與未來優化建議 (Next Steps / Backlog)

給接手的 Agent 或開發者，以下為可進一步擴充或深化的潛在需求方向：

### 優先級 1：單人電腦對戰 (PVE AI Mode)
- [ ] **AI 電腦對手 (Simple / Smart Billiards AI)**：
  - 目前雙人對戰模式需要兩位真人輪流出桿。可新增「🤖 與 AI 機器人對戰」選項。
  - AI 演算法實作：利用現有的假想球計算器 (`aimAtGhostBall`) 與顆星反射射線，評估可進球路線並自動計算瞄準角度與力道。
  - 提供兩種難度：幼幼班（偶爾會有些微誤差角度）、神射手（精準走位與做球）。

### 優先級 2：球桌外觀自訂與視覺特效 (Cosmetics & Polish)
- [ ] **球呢布顏色自訂選單**：
  - 支援傳統綠布（Classic Green）、賽事藍布（Tournament Blue）、英倫酒紅（Burgundy Red）。
- [ ] **進球落袋與慶祝粒子特效**：
  - 子球進袋瞬間加入金色微粒噴發特效與清脆祝賀音效，增添成就感。
- [ ] **音量調節與靜音按鈕**：
  - 在導覽列或右上角加入音量拉桿與一鍵靜音切換按鈕。

### 優先級 3：花式解球與更多經典局 (Trick Shots & Challenges)
- [ ] **挑戰特訓模式擴充**：
  - 經典一桿清台殘局挑戰。
  - 花式跳球撞雙球、雙星扎桿繞過兩顆球等高難度關卡。
- [ ] **自訂排球模式 (Sandbox Custom Rack)**：
  - 允許玩家自由在球桌上擺放任意數量的子球與障礙球，並匯出/分享關卡排局給其他小朋友挑戰。

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
