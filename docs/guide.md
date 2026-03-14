# Agentic Engineering 實戰指南

> 基於 SysLS 的文章 *"How To Be A World-Class Agentic Engineer"* 核心觀點，結合實戰經驗重新編寫。
>
> 這份文件講**觀念**。實際操作請用 `skill/` 裡的 optimizer，丟給 AI 執行就好。

---

## 你的 Agent 為什麼表現不穩定

不是你的 harness 不對、不是你的 plugin 不夠、不是你的 terminal 有問題。

問題幾乎永遠是同一個：**你餵給 agent 太多它不需要的資訊。**

---

## 觀念一：Context Is Everything

Agent 的表現直接取決於它在執行任務時「腦袋裡裝了什麼」。裝太多 = 表現變差。

你要它寫一個 hangman game，結果它腦袋裡同時塞滿了「26 個 session 前的 memory」、「怎麼發通知訊息」、「HTML 的 CSS 偏好」。它當然表現不好。

Context bloat（上下文膨脹）的常見來源：過多的 plugin 注入、巨型 CLAUDE.md、memory system 殘留、不分場合載入所有規則。

**核心動作：只給 agent 完成當前任務所需的資訊，其他一概不給。**

---

## 觀念二：CLAUDE.md 是 Router，不是百科全書

原文最重要的一句實作建議：

> Treat your CLAUDE.md as a logical, nested directory of where to find context given a scenario and an outcome. It should be as barebones as possible, and only contain the IF-ELSE of where to go to seek the context.

**CLAUDE.md 應該長這樣：**

```
啟動流程 → 讀哪些檔案來恢復 context
安全底線 → 絕對不能做的事（永遠生效）
路由規則 → 寫程式時讀 A，發訊息時讀 B，寫內容時讀 C
```

**不應該長這樣：**

```
啟動流程（50 行）+ 任務流程（80 行）+ 寫程式規則（100 行）
+ 測試規則（60 行）+ 發送規則（40 行）+ HTML 偏好（30 行）
+ 寫作風格（50 行）+ 經驗教訓（200 行）= 600+ 行
```

第一種 CLAUDE.md 大約 50 行。Agent 寫程式時只多讀一份 coding-rules.md。
第二種 CLAUDE.md 600 行。Agent 不管做什麼都要先吞 600 行。

差異是 context 的乾淨程度，直接影響 agent 的專注力和表現。

---

## 觀念三：Rules vs Skills — 不要搞混

- **Rule = 偏好（preference）**：「不要用 any type」「變數名用 camelCase」「覆蓋前要問我」
- **Skill = 配方（recipe）**：「部署的完整步驟是 1→2→3→4→5」「收到截圖的處理流程」

Rule 告訴 agent「注意什麼」。Skill 告訴 agent「怎麼做某件事」。

混在一起 → agent 搞不清楚哪些是限制、哪些是流程。

---

## 觀念四：疊加 → 矛盾 → 清理，是自然循環

你會不斷加 rule。加到某個程度，它們會開始互相矛盾、產生冗餘、或者過時。然後 agent 表現又開始變差。

**這不是 bug，這是必然的。** 原文叫它 "spa day"：定期讓 agent 掃描你所有的 rules 和 skills，找出矛盾和冗餘，提議精簡。

循環就是：**用 → 加 rule → 表現變好 → 繼續加 → 表現變差 → 清理 → 表現恢復 → 重複**。

---

## 觀念五：Compaction 後的 Context 恢復

長對話中 Claude 會壓縮（compaction）前面的對話。壓縮後它可能「忘記」重要的 context。

解法很簡單：在 CLAUDE.md 的啟動流程中寫清楚「compaction 後重新讀哪些檔案」。這樣每次壓縮後它都知道怎麼恢復狀態。

---

## 觀念六：不要焦慮，不需要追新工具

> If something truly is ground-breaking and extended agentic use-cases in a meaningful way, it will be incorporated into the base products of the foundation companies in due time.

Skills、memory、subagents——這些都是社群先發明，然後被 Claude / Codex 官方吸收的。真正有用的功能不需要你去找，它會自己來。

你只需要定期更新 CLI，看 release notes。精力花在理解自己的工作流程、觀察 agent 犯錯、轉化成 rules 和 skills 就夠了。

---

## 怎麼用這個 Repo

1. 把 `skill/agentic-setup-optimizer/SKILL.md` 放進你的 skills 目錄
2. 跟 agent 說「優化我的設定」
3. 它會自動掃描 → 診斷 → 建議 → 你確認後執行

就這樣。概念你已經懂了，剩下的讓 AI 做。

---

## 附錄

本指南基於 SysLS 的 *"How To Be A World-Class Agentic Engineer"*。原文還涵蓋了 sycophancy 利用（對抗式 agent 架構）、task contracts、永不停機的 agent 等進階主題，有興趣可以找原文來看。
