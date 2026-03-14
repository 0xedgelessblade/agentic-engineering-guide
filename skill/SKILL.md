# Agentic Setup Optimizer

> 互動式 Agent 設定優化器。掃描使用者現有的 CLAUDE.md / rules / skills，診斷問題，產出優化建議，執行重組。

## 觸發條件

當使用者說以下任何一種時觸發：
- 「優化我的設定」、「優化 CLAUDE.md」、「檢查我的 agent 設定」
- 「setup optimizer」、「optimize my setup」
- 「我的 agent 表現變差了」、「為什麼 agent 越來越笨」
- 「清理 rules」、「整理 skills」、「spa day」

## 核心原則

在執行任何操作前，先理解這些原則：

1. **Context is everything**：agent 的表現取決於它能看到的資訊。過多 = 表現變差
2. **CLAUDE.md 是導航表（router）**：只放路由邏輯和安全底線，其他規則按需載入
3. **Rules 編碼偏好，Skills 編碼配方**：不要混淆這兩者
4. **疊加 → 矛盾 → 清理是自然循環**：不是出了問題，是該清理了

## 執行流程

### Phase 1：掃描現狀

1. **找到 CLAUDE.md**：
   ```
   先檢查 .claude/CLAUDE.md
   再檢查根目錄 CLAUDE.md
   再檢查 AGENT.md
   ```
   讀取完整內容。

2. **找到 rules 目錄**：
   ```
   檢查 _context/rules/
   檢查 rules/
   檢查 .rules/
   ```
   列出所有 .md 檔案，每個都讀取。

3. **找到 skills 目錄**：
   ```
   檢查 _context/skills/
   檢查 skills/
   檢查 .skills/
   ```
   列出所有 SKILL.md，每個都讀取。

4. **找到 lessons / memory**：
   ```
   檢查 _context/lessons-learned.md
   檢查 CLAUDE.local.md
   檢查 memory/ 或 .memory/
   ```

5. 如果以上都找不到，告訴使用者：「你目前沒有任何設定，建議從模板開始。」並提供模板。

### Phase 2：診斷

掃描完所有檔案後，進行以下診斷。每項都要有具體的引述（哪個檔案、哪一段）：

#### 2.1 Context Bloat 檢查
- 計算 CLAUDE.md 的行數
  - > 100 行：⚠️ 可能太長，建議拆分
  - > 300 行：🔴 嚴重膨脹，必須拆分
- 計算所有 rules + skills 的總行數
- 判斷：「agent 在開始任何任務前，需要讀多少資訊？」

#### 2.2 路由架構檢查
- CLAUDE.md 是否有「按需載入」的路由邏輯？
- 還是所有規則都塞在同一個檔案裡？
- 有沒有規則是「不管什麼任務都會被載入，但其實只有特定場景需要」的？

#### 2.3 矛盾檢查
- 掃描所有 rules，找出互相矛盾的指令
- 常見矛盾模式：
  - 「永遠做 X」 vs 「某些情況下不要做 X」
  - 同一件事在不同檔案有不同說法
  - 舊的 rule 和新的 rule 打架

#### 2.4 冗餘檢查
- 同樣的規則是否出現在多個檔案裡？
- 有沒有 rule 是模型已經自然會遵守的（不再需要特別強調）？
- 有沒有 rule 是太過具體的一次性指令，不適合作為永久規則？

#### 2.5 Rule vs Skill 混淆檢查
- 有沒有 rule 檔案裡放了完整的執行步驟（應該是 skill）？
- 有沒有 skill 檔案裡放了偏好設定（應該是 rule）？

### Phase 3：報告

用以下格式向使用者報告診斷結果：

```markdown
## 設定健康報告

### 概況
- CLAUDE.md 行數：XXX
- Rules 檔案數：X，總行數：XXX
- Skills 檔案數：X，總行數：XXX
- Agent 啟動時需讀取的 context 量：約 XXXX 字

### 發現的問題

#### 🔴 嚴重（立即處理）
1. ...（具體問題 + 在哪個檔案 + 引述）

#### ⚠️ 建議改善
1. ...

#### ✅ 做得好
1. ...

### 建議的優化方案
1. ...（具體步驟）
```

### Phase 4：執行優化（使用者確認後）

**絕對不要在使用者確認前修改任何檔案。**

向使用者展示報告後，詢問：
- 「你想要我執行哪些優化？」
- 「有沒有哪些建議你不同意？」

使用者確認後：

1. **備份**：先把所有要修改的檔案備份到 `_backup/YYYY-MM-DD/`
2. **執行**：按照確認的方案修改
3. **驗證**：修改完成後，重新跑一次 Phase 2 的診斷，確認問題已解決
4. **報告**：向使用者展示 before/after 對比

## 目標架構參考

優化的終極目標是讓使用者的設定趨近以下架構。這是一個經過實戰驗證的範例：

```
CLAUDE.md（~60 行，導航 + 不可跳過的安全規則）
│
├── 永遠載入：
│   ├── 寫入不可跳過的步驟 + 紅旗清單 + 絕對禁止（安全底線，不能冒險不讀）
│   └── lessons-learned.md（只放核心通病摘要）
│
├── 按需導航（IF-ELSE 路由）：
│   ├── 收到任務 → 讀 _context/rules/task-flow.md
│   ├── 寫程式 → 讀 _context/rules/coding-rules.md
│   ├── 發通知訊息 → 讀 _context/rules/messaging-rules.md
│   ├── 輸出 HTML → 讀 _context/rules/html-preferences.md
│   ├── 寫對外內容 → 讀 voice-style.md + about-me.md
│   └── 查系統設定 → 讀 system-config.md
│   └── 以此類推，按功能增加
```

### 架構重點

1. **CLAUDE.md ≤ 60 行**：只放「每次都需要」的安全底線 + 路由表
2. **安全底線不能拆出去**：寫入驗證、紅旗清單、絕對禁止 → 永遠在 CLAUDE.md 裡，不依賴按需載入
3. **路由用明確觸發條件**：「寫程式 → 讀 X」，不要用「如果你覺得相關就讀 X」
4. **每個 rule 檔案職責單一**：一個檔案管一個場景，不要混搭
5. **lessons-learned 只讀摘要**：核心通病 4-5 行就夠，完整報告放 archive，啟動時不讀

### 對應的目錄結構

```
workspace/
├── .claude/
│   └── CLAUDE.md              ← 路由表 + 安全底線（自動載入，~60 行）
│
└── _context/
    ├── working-rules.md       ← 規則架構導覽（指向各 rule 的地圖）
    ├── lessons-learned.md     ← 經驗教訓摘要（核心通病 + 一行摘要表）
    ├── lessons-learned-archive.md  ← 完整教訓報告（不在啟動時讀）
    ├── rules/
    │   ├── task-flow.md       ← 收到任務時的流程
    │   ├── coding-rules.md    ← 寫程式的規則
    │   ├── messaging-rules.md ← 訊息發送方案
    │   └── html-preferences.md ← HTML 輸出風格
    ├── voice-style.md         ← 寫作風格（對外內容時用）
    └── about-me.md            ← 個人資料（對外內容時用）
```

## 常見優化操作

### 操作 A：拆分巨型 CLAUDE.md
1. 識別 CLAUDE.md 中可以按需載入的區塊
2. 每個區塊搬到 `_context/rules/` 底下的獨立檔案
3. 在 CLAUDE.md 中用路由規則替代（「做 X 時讀 Y」）
4. CLAUDE.md 只保留：啟動流程 + 安全底線 + 路由規則 + 個人偏好

### 操作 B：合併重複規則
1. 找出所有重複的規則
2. 保留最完整的版本
3. 刪除其他版本
4. 確認路由指向正確的檔案

### 操作 C：建立缺失的架構
如果使用者完全沒有設定：
1. 建立 `_context/` 目錄
2. 建立 `_context/rules/` 目錄
3. 從模板建立最小的 CLAUDE.md
4. 從模板建立 task-flow.md

### 操作 D：Rule / Skill 分類修正
1. 把 rule 檔案中的「完整執行步驟」搬到 skill
2. 把 skill 檔案中的「偏好設定」搬到 rule

## 注意事項

- 每次只做一件事，不要一次大改
- 每步都確認使用者同意
- 修改前一定備份
- 修改後一定驗證
