# CLAUDE.md 模板

> 這是一個最小可用的 CLAUDE.md 模板。只放三樣東西：啟動流程、安全底線、路由規則。
> 所有其他規則都放在 `_context/rules/` 底下，按需載入。

---

## 啟動流程（每次 session 開始或 compaction 後執行）

1. **讀取環境設定**：讀取 `_context/machine-config.md`，確認當前機器和路徑
2. **讀取規則導覽**：讀取 `_context/working-rules.md`，了解規則架構
3. **讀取核心教訓**：讀取 `_context/lessons-learned.md` 的前 10 行（核心通病）
4. **驗證工作目錄**：`ls` 當前 workspace，確認 `_context/` 目錄存在。不存在 = 錯的目錄，立即停止

## 安全底線（永遠生效，不依賴按需載入）

### 寫入前驗證
1. `ls` 目標位置 — 有同名檔案嗎？
2. 有 → `Read` 目標檔案 — 看內容
3. 目標檔案有實質內容 → **不能覆蓋，必須合併或問使用者**

### 絕對禁止
- 未經確認不刪除任何檔案
- 未經確認不覆蓋任何現有檔案
- 不在沒有依據的情況下編造數據

## 路由規則（按需載入）

收到任務時，根據任務類型讀取對應的規則檔案：

| 場景 | 讀取 |
|------|------|
| 收到任何任務 | `_context/rules/task-flow.md` |
| 寫程式 | `_context/rules/coding-rules.md` |
| 測試失敗 | `_context/rules/test-failing-rules.md` |
| 寫對外內容 | `_context/voice-style.md` |
| （自行擴充）| （自行擴充）|

## 個人偏好

> 在使用過程中逐步填入。例如：
> - 預設語言：繁體中文
> - 輸出格式偏好：.md
> - （你的偏好...）
