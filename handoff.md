# 小朋友英文大冒險 — Handoff 文件

> 給未來 Claude session 的快速上手指南。每次大改動後請更新此檔案。
> 最後更新：2026-05-31

---

## 專案基本資訊

| 項目 | 內容 |
|------|------|
| 唯一檔案 | `C:\Users\Chi\Desktop\claude-main\index.html` |
| 本機伺服器 | `http://localhost:3000`（.claude/launch.json 已設定） |
| 語言 | 單檔 HTML + CSS + JavaScript（無框架、無 npm） |
| 總行數 | ~5400+ 行 |
| 開發者 | Chi（程式新手，繁體中文溝通） |
| GitHub repo | `https://github.com/elain-chi/englishwords-adventure` |
| 公開網址 | `https://elain-chi.github.io/englishwords-adventure/` |
| 部署方式 | GitHub Pages，main branch 自動部署，push 後幾分鐘生效 |

---

## 架構概覽

```
index.html
├── <style>          全部 CSS
├── <body>           多個 screen div（同時只顯示一個）
│   ├── screen-start     首頁
│   ├── screen-char      角色選擇
│   ├── screen-level     關卡選擇
│   ├── screen-map       地圖（格子探索）
│   ├── screen-learn     學習單字
│   ├── screen-quiz      測驗
│   ├── screen-result    測驗結果
│   ├── screen-win       勝利畫面
│   └── event-modal      事件彈窗（飯糰、西瓜、boss 戰都用這個）
└── <script>         全部邏輯
```

### 切換畫面
```javascript
showScreen('level')   // 顯示關卡選擇
showScreen('map')     // 顯示地圖
```

### 全域狀態
```javascript
gameState.player       // 角色名、等級、character key
gameState.session      // 本局：wordGroups、hammerCount、animalFed、wrongWords…
gameState.progress     // 存檔：players[].learnedWordIds、sessionHistory…
```

---

## 關鍵函式位置（行號僅供參考，可能隨編輯漂移）

| 函式 | 約行號 | 說明 |
|------|--------|------|
| `showScreen(id)` | ~1452 | 切換畫面，level 畫面有重置選取邏輯 |
| `buildSession()` | ~1712 | 組本局 8/12 個單字（L1=2個/組，L2+=3個/組） |
| `startLearningPhase(groupIdx)` | ~1770 | 開始學習某組單字 |
| `renderWordCard()` | ~1835 | 顯示單字卡（含 400ms auto-speak） |
| `nextWordCard()` | ~2065 | 下一張單字卡，會呼叫 `_cancelTTS()` |
| `showPreQuizTransition()` | ~2100 | 測驗前提示，動態顯示「兩個/三個」 |
| `quizComplete()` | ~2195 | 測驗結束，複習模式有特殊分支 |
| `startReviewSession(wordIds, level, profileId)` | ~2250 | 從遊戲中心「複習這一局」進入 |
| `bossBattleEnd()` | ~3800 | 魔王結束，存 sessionHistory |
| `pmShowSession(playerIdx, sessionIdxStr)` | ~4700 | 遊戲中心顯示某局單字 ✅/❌ |
| `exportWordDatabase()` | ~500 | 匯出 CSV |
| `goHome()` | ~1440 | 回首頁（resetGame + initStart） |

### TTS 相關
```javascript
// 全域 ~1455
let _autoSpeakTimer = null;
let _ttsGeneration  = 0;
function _cancelTTS() { ... }   // 立即中斷發音
function _speakQueue(items, onDone) { ... }  // 有 generation counter 防殘留
```

### 動物養成相關
```javascript
// 全域 ~3005
const ANIMAL_LEVEL_EMOJIS = { penguin:[...], lion:[...], ... };
const ANIMAL_LEVEL_NAMES  = ['蛋蛋期 🌱', ...];
const LEVEL_THRESHOLDS    = [0, 3, 9, 18, 28, 38, 48, 58, 68];

function getAnimalLevel()              // 回傳 0–8
function getAnimalStageEmoji(char, level)
function getAnimalEmoji()
```

---

## 遊戲流程

```
首頁 → 角色選擇 → 關卡選擇
  → 地圖（格子探索）
    ├── 學習格：學單字(2或3個) → 測驗 → 飯糰popup → 回地圖
    ├── 挑戰格：測驗 → 西瓜popup → 回地圖
    └── 魔王格：閃電回合(3題) → boss戰 → 勝利/失敗
```

---

## 單字庫格式

```javascript
const WORDS = [
  {
    id: 'L1_S1_001',
    english: 'mom',
    chinese: '媽媽',
    emoji: '👩',
    level: 1,
    sentences: [
      { en: 'I love mom.', zh: '我愛媽媽。' },
      { en: 'Mom is happy.', zh: '媽媽很開心。' }
    ]
  },
  // ...
];
```

---

## 各等級完成狀態

| 等級 | 單字數 | 句子 | 狀態 |
|------|--------|------|------|
| L1 | 175 | 350（每字2句） | ✅ 完成 |
| L2 | — | ≤5字/句 | ❌ 待製作 |
| L3 | — | ≤6字/句 | ❌ 待製作 |
| L4 | — | ≤7字/句 | ❌ 待製作 |
| L5 | — | ≤10字/句 | ❌ 待製作 |
| L6 | — | ≤10字/句 | ❌ 待製作 |

L1 Word ID 場景前綴：S1 家庭、S6 動作、BD 身體、NM 數字、CL 顏色、AN 動物、FD 食物、VB 動詞、AJ 形容詞、TH 物品、PL 場所、NT 自然、TR 交通、EX 補充

---

## 動物成長系統

| 階段 | 累積飯糰 | 顯示 |
|------|----------|------|
| 蛋蛋期 | 0 | 🥚 |
| 小寶寶 | 3 | 角色幼體 |
| 成長期 | 9 | 角色成體 |
| 成熟期 | 18 | 角色特殊（兔🐇🥕，狐🦊🍗） |
| ⭐ 一星 | 28 | 成熟期 + ⭐ |
| ⭐⭐ 二星 | 38 | 成熟期 + ⭐⭐ |
| ⭐⭐⭐ 三星 | 48 | 成熟期 + ⭐⭐⭐ |
| ⭐⭐⭐⭐ 四星 | 58 | 成熟期 + ⭐⭐⭐⭐ |
| ⭐⭐⭐⭐⭐ 五星 | 68 | 成熟期 + ⭐⭐⭐⭐⭐ |

---

## 最近完成的改動（2026-05-31）

### 斧頭系統（全新功能）
1. **槌子→斧頭鍛造** — 累積 10 個槌子時，回到地圖觸發鍛造詢問視窗（`checkForgeAxe()`）
2. **鍛造動畫** — 10 個槌子繞圓飛向中心消失 → 大斧頭出現 + 星星特效（`forgeAxe()`）
3. **HUD 顯示斧頭** — 地圖頁左側顯示 🪓×N，槌子右側
4. **斧頭存檔** — `profile.savedAxeCount`，魔王關結束後保存

### 魔王攻擊系統
5. **斧頭攻擊詢問** — 踩到魔王格且斧頭≥3，彈出是/否視窗（`_offerAxeAttack()`）
6. **斧頭攻擊動畫** — 3 把斧頭依序從魔王頭上往下砍，魔王左右搖晃+變色（`bossShakeLR`）
7. **攻擊音效** — 新設計「咚」聲：300Hz→120Hz 主音 + 800Hz 撞擊瞬間 + 帶通噪音
8. **魔王減血** — 使用斧頭後魔王減 2 滴血，只需過 4 關（正常 6 關）

### Android 返回鍵
9. **返回鍵攔截** — `history.pushState` + `popstate`，首頁不攔截，遊戲中彈出確認視窗
10. **自訂對話框** — 顯示寵物圖示 + 玩家名稱，「是→存檔離開」/ 「否→繼續玩」

### 新增 CSS 動畫
- `@keyframes forgeHammer` — 槌子飛向中心
- `@keyframes forgeAxeAppear` — 斧頭出現
- `@keyframes bossShakeLR` — 魔王左右搖晃+變色（攻擊時用）

### 關鍵新函式
| 函式 | 說明 |
|------|------|
| `checkForgeAxe()` | 地圖返回時檢查是否有 10 個槌子可鍛造 |
| `forgeAxe()` | 執行鍛造動畫 |
| `_offerAxeAttack(available)` | 詢問是否用斧頭攻擊魔王 |
| `_showAxeAttackAnimation(available)` | 斧頭攻擊動畫，結束後進入 4 關魔王戰 |
| `_playAxeHitSound()` | 咚聲音效（300Hz 範圍，喇叭可聽到） |
| `_initBossBattle(available, totalRounds)` | 統一初始化魔王戰（4 或 6 關） |
| `_showBackButtonDialog()` | Android 返回鍵自訂對話框 |
| `_backBtnConfirm(exit)` | 處理是/否選擇 |

## 最近完成的改動（2026-05-19 這次 session）

1. **飯糰/西瓜彈跳視窗置中** — 改用 `showEventOverlay()` 取代黏底的 insertAdjacentHTML
2. **動物養成升級條件** — 改為非線性 LEVEL_THRESHOLDS，成熟期特殊 emoji
3. **L1 每局學 8 字** — `wordsPerGroup = level===1 ? 2 : 3`
4. **遊戲中心學習歷程** — 下拉選關卡、顯示 ✅/❌、「複習這一局」按鈕
5. **TTS 取消機制** — `_cancelTTS()` + generation counter，防止發音殘留
6. **PreQuiz 動態文字** — L1 顯示「兩個單字/兩題全對」
7. **閃電回合中文字** — 圖片下方加 `${word.chinese}`
8. **已完成關卡可重新點選** — `showScreen('level')` 時重置選取狀態
9. **餵食按鈕文字** — 學習關「🍙 餵飯糰！」，挑戰關「🍉 餵西瓜！」

## 文件與工作流程整理（2026-05-21）

1. **建立 `handoff.md`** — 放在專案根目錄，跨裝置交接用
2. **建立全域 `CLAUDE.md`** — `C:\Users\Chi\.claude\CLAUDE.md`，所有新專案都適用
3. **「存檔/收工」觸發自動更新** — 說存檔或收工時自動更新 handoff.md 和記憶檔
4. **確認記憶系統範圍** — 記憶資料夾是專案路徑專屬，搬移資料夾後需重新存檔補回
5. **確認開新專案流程** — `≡` → `File` → `Open File` → 選取資料夾
6. **建立 `工作流程筆記.md`** — 操作流程速查，涵蓋開新專案、切換專案、跨裝置搬移
7. **補充 GitHub 資訊** — repo 和 GitHub Pages 網址已補進 handoff.md

---

## 已知限制 / 注意事項

- `sessionHistory` 舊存檔（2026-05-17 前）沒有 `words[]` 欄位，`pmShowSession` 會顯示「舊存檔，無單字記錄」
- `preview_click` 有時無法點到 `event-modal` 裡的按鈕，改用 `preview_eval` 直接呼叫函式測試
- 本機伺服器需先在 Claude Code 啟動（Run 按鈕或 launch.json）

---

## ⚠️ 存檔 / 收工 強制流程（每次都要執行）

Chi 說「存檔」或「收工」時，**必須**依序完成以下步驟：

1. 更新 `handoff.md`（本次做了什麼）
2. 更新 memory 記憶檔（`project_game_state.md`）
3. **推上 GitHub**（這個專案每次存檔都要推）：
   ```bash
   git add index.html handoff.md
   git commit -m "收工存檔：[本次改動摘要]"
   git push origin master
   git checkout main
   git merge master --no-edit
   git push origin main
   git checkout master
   ```
   > ⚠️ 本機開發在 `master`，GitHub Pages 部署 `main`，每次都要同步兩個分支

---

## 下次開始時的建議步驟

1. 讀此 `handoff.md` 了解現況
2. 讀 `.claude/projects/.../memory/project_game_state.md` 確認記憶一致
3. 用 `http://localhost:3000` 確認本機伺服器是否已啟動
4. 若要加 L2 單字庫，先確認 Chi 對 L1 沒有修改需求
