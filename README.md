# Agentic Engineering 實戰指南

> 讓你的 Claude / Codex 設定從「一團亂」變成「有結構、可維護、高效能」。

基於 SysLS 的 *"How To Be A World-Class Agentic Engineer"* 核心觀點，結合實戰經驗重新編寫。

**這不是又一篇觀念文。** 這個 repo 包含一個可以直接丟給 AI 執行的 skill，讓它幫你掃描、診斷、優化你的設定。

---

## 你的 Agent 為什麼表現不穩定

不是你的 harness 不對、不是你的 plugin 不夠、不是你的 terminal 有問題。

問題幾乎永遠是同一個：**你餵給 agent 太多它不需要的資訊。**

你要它寫一個 hangman game，結果它腦袋裡同時塞滿了「26 個 session 前的 memory」、「怎麼發通知訊息」、「HTML 的 CSS 偏好」。它當然表現不好。

Context bloat（上下文膨脹）的常見來源：過多的 plugin 注入、巨型 CLAUDE.md、memory system 殘留、不分場合載入所有規則。

**核心動作：只給 agent 完成當前任務所需的資訊，其他一概不給。**

---

## 核心架構：CLAUDE.md 是 Router，不是百科全書

這是整個指南最重要的觀念。原文最關鍵的一句話：

> Treat your CLAUDE.md as a logical, nested directory of where to find context given a scenario and an outcome. It should be as barebones as possible, and only contain the IF-ELSE of where to go to seek the context.

翻譯：**CLAUDE.md 應該是一個路由器（router），不是一本百科全書。**

### ❌ 大多數人的 CLAUDE.md 長這樣

```
# CLAUDE.md（600+ 行，全部塞在一起）

啟動流程（50 行）
任務流程（80 行）
寫程式規則（100 行）
測試規則（60 行）
訊息發送規則（40 行）
HTML 偏好（30 行）
寫作風格（50 行）
經驗教訓（200 行）
```

結果：不管做什麼任務，agent 都要先吞 600 行。寫程式時腦袋裡有訊息發送規則，發訊息時腦袋裡有寫程式規則。全部互相干擾。

### ✅ 應該長這樣：導航化 + 規則模組化

```
CLAUDE.md（~60 行，導航 + 不可跳過的安全規則）
│
├── 永遠載入：
│   ├── 啟動流程（短，恢復 context 用）
│   ├── 寫入前驗證 + 紅旗清單 + 絕對禁止（安全底線，不能冒險不讀）
│   └── lessons-learned.md（只讀核心通病摘要）
│
├── 按需導航（IF-ELSE 路由）：
│   ├── 收到任務 → 讀 _context/rules/task-flow.md
│   ├── 寫程式 → 讀 _context/rules/coding-rules.md
│   ├── 發通知訊息 → 讀 _context/rules/messaging-rules.md
│   ├── 輸出 HTML → 讀 _context/rules/html-preferences.md
│   ├── 寫對外內容 → 讀 voice-style.md + about-me.md
│   └── 查系統設定 → 讀 system-config.md
```

對應的檔案目錄結構：

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

### 為什麼 router 模式有效

| | 百科全書模式 | Router 模式 |
|---|---|---|
| CLAUDE.md 大小 | 600+ 行 | ~60 行 |
| 寫程式時的 context | 600 行全吃 | 60 行 + coding-rules.md |
| 發訊息時的 context | 600 行全吃 | 60 行 + messaging-rules.md |
| 修改某個規則 | 在巨大檔案裡翻找 | 直接改對應的檔案 |
| 規則互相矛盾 | 很容易，因為全混在一起 | 不容易，各檔案職責單一 |
| Compaction 後恢復 | 600 行壓縮後丟失嚴重 | 60 行路由表很難被壓壞 |

### 架構重點

1. **CLAUDE.md ≤ 60 行**：只放「每次都需要」的安全底線 + 路由表
2. **安全底線不能拆出去**：寫入驗證、紅旗清單、絕對禁止 → 永遠在 CLAUDE.md 裡
3. **路由用明確觸發條件**：「寫程式 → 讀 X」，不要用「如果你覺得相關就讀 X」
4. **每個 rule 檔案職責單一**：一個檔案管一個場景，不要混搭
5. **lessons-learned 只讀摘要**：核心通病 4-5 行就夠，完整報告放 archive

---

## 其他核心觀念

### Rule ≠ Skill，不要搞混

- **Rule = 偏好（preference）**：「不要用 any type」「變數名用 camelCase」「覆蓋前要問我」
- **Skill = 配方（recipe）**：「部署的完整步驟是 1→2→3→4→5」「收到截圖的處理流程」

Rule 告訴 agent「注意什麼」。Skill 告訴 agent「怎麼做某件事」。混在一起 → agent 搞不清楚哪些是限制、哪些是流程。

### 定期清理是必要的

你會不斷加 rule。加到某個程度，它們會開始互相矛盾、產生冗餘、或者過時。然後 agent 表現又開始變差。

**這不是 bug，這是必然的。** 原文叫它 "spa day"。循環就是：**用 → 加 rule → 表現變好 → 繼續加 → 表現變差 → 清理 → 表現恢復 → 重複**。

### Compaction 後的 Context 恢復

長對話中 Claude 會壓縮前面的對話。壓縮後它可能「忘記」重要的 context。解法：在 CLAUDE.md 的啟動流程中寫清楚「compaction 後重新讀哪些檔案」。

### 不需要追新工具

Skills、memory、subagents——這些都是社群先發明，然後被 Claude / Codex 官方吸收的。真正有用的功能不需要你去找，它會自己來。定期更新 CLI，看 release notes 就夠了。

---

## 使用方式

1. 把 `skill/` 裡的 SKILL.md 放進你的 skills 目錄
2. 跟 agent 說：**「優化我的設定」**
3. AI 自動掃描 → 診斷 → 建議 → 你確認後執行

概念你讀完了，剩下的讓 AI 做。

更多細節請看 → [`docs/guide.md`](docs/guide.md)

---

## Repo 架構

```
agentic-engineering-guide/
│
├── docs/
│   ├── guide.md            # 📖 觀念篇（完整版）
│   └── guide.html          #    HTML 版本（瀏覽器閱讀）
│
├── skill/
│   └── SKILL.md            # 🤖 丟給 AI 執行：掃描設定 → 診斷 → 優化
│
├── templates/              # 📋 起手模板
│   ├── CLAUDE.md           #    最小可用 CLAUDE.md（router 模式）
│   └── rules/
│       └── task-flow.md    #    任務流程規則
│
└── examples/
    └── common-mistakes.md  # 💡 常見錯誤 + 解法
```

## 致謝

核心觀點源自 [SysLS](https://www.reddit.com/user/SysLS/) 的文章 *"How To Be A World-Class Agentic Engineer"*

## License

MIT
