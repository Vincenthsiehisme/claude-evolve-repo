# Skill 進化 Tag 集合（單一真實來源）

> 這是 skill-summary 自動分類與 skill-evolve 診斷規則的共同依據。
> 修改本檔等於修改兩支 skill 的行為，請審慎。

---

## 設計原則

- **MECE**：互斥（一個錯誤對應一個 tag）+ 完全窮盡（涵蓋 skill 失效的所有面向）
- **層次化**：依「skill 失效的根本面向」分四層 + 1 meta 層
- **命名規範**：`動詞-名詞` 或 `名詞-狀態`，全小寫、連字號

---

## 7 個固定 Tag

### 觸發層（Triggering）

#### `trigger-miss`
**含義**：skill 該觸發沒觸發。
**典型症狀**：使用者描述了該 skill 的場景，但 Claude 沒進入該 skill。
**對應的優化方向**：description 觸發詞要補。
**範例**：使用者說「erp-schema 表怎麼設計」，erp-schema 沒被觸發 → 觸發詞缺「表怎麼設計」這類變體。

#### `trigger-overreach`
**含義**：skill 不該觸發卻觸發了。
**典型症狀**：Claude 進入某 skill，但實際上該走鄰居 skill。
**對應的優化方向**：description 要加 DO NOT trigger 邊界。
**範例**：使用者問「商品搜不到」，skill-search 誤觸發（該走 search-map） → DO NOT trigger 缺對 search-map 的排除。

---

### 理解層（Comprehension）

#### `intent-misread`
**含義**：skill 觸發了、reference 也夠，但**誤解使用者意圖**導致答錯方向。
**典型症狀**：Claude 跑了正確 skill、走了正確流程，但答非所問。
**對應的優化方向**：SKILL.md 主流程需加意圖判讀步驟（Step 0 模式）。
**範例**：prd-writer 觸發後直接跳到模板填寫，沒判讀使用者其實在「模糊探索」階段 → Step 0 缺意圖判讀。

---

### 內容層（Knowledge）

#### `coverage-gap`
**含義**：該知道的內容沒寫在 reference（含「重複踩同一坑」）。
**典型症狀**：Claude 給的答案缺了某個關鍵資訊；或同樣的坑踩了第二次以上。
**對應的優化方向**：reference 要補新內容、或 Gotchas 要補新條目。
**範例**：erp-schema 沒指出 tppdm 跟 bookm 的 FK 是 tpno → references/fk-rules.md 漏這對配對。

> **注意**：原本提案的 `gotcha-repeat` 已併入此 tag——重複踩坑就是 reference 不夠，根因相同。skill-evolve 會基於累積條數自動偵測「這條 coverage-gap 已經第 N 次」。

#### `coverage-stale`
**含義**：reference 有寫但內容過期或錯誤。
**典型症狀**：Claude 引用了 reference 但答案不對（不是漏寫，是寫錯了）。
**對應的優化方向**：reference 要更新。
**範例**：task-dispatcher 的 squad-scope.json 寫 S5 提供導領資料給 S4，實際是 S4 提供給 S5 → 要更正。

---

### 輸出層（Output）

#### `output-drift`
**含義**：觸發、理解、知識都對，但輸出格式跑掉。
**典型症狀**：邏輯內容正確，但結構/語氣/排版不對。
**對應的優化方向**：SKILL.md 的輸出規格要強化、或加範本檔。
**範例**：critical-reviewer 的整合段退化為摘要列點，沒做交叉比對 → 輸出規格需強調「找獨立收斂與張力」。

---

### 協作層（Orchestration）

#### `routing-error`
**含義**：子 skill 沒被父 skill 正確呼叫，或 skill 鏈失效。
**典型症狀**：父 skill（如 story-orchestrator）跑完了，但沒呼叫該呼叫的子 skill（如 flow-drawer）。
**對應的優化方向**：父 skill 的協作邏輯要強化。
**範例**：story-orchestrator 該呼叫 api-designer 但跳過了 → orchestrator 主流程需明寫呼叫順序。

---

### Meta 層（Boundary）

#### `scope-overlap`
**含義**：跨 skill 範疇衝突（不屬於單一 skill 的內部錯誤）。
**典型症狀**：兩個 skill 同時被觸發、或互搶觸發、或職責不清楚使用者不知該用哪個。
**對應的優化方向**：兩個 skill 的 description 邊界要切清、或考慮合併/拆分。
**範例**：skill-evolve 跟 skill-review 都被使用者誤呼叫做「審 skill」 → 兩支的 description 必須明寫分工。

**特殊存放路徑**：`logs/cross-skill/scope-overlap.md`（不放進單一 skill 目錄）
**特殊欄位**：每筆紀錄必須含 `Skills involved` 欄，列出涉及的 skill 對（字母排序）。

---

## Tag 自動分類對照（給 skill-summary 用）

skill-summary 既有 Step 0 已判讀錯誤類型 A/B/C/D，可推 tag：

| Step 0 類型 | 推薦 tag |
|---|---|
| A. 觸發失敗 | trigger-miss / trigger-overreach |
| B. 過程踩坑 | coverage-gap / intent-misread / coverage-stale |
| C. 輸出不對 | output-drift |
| D. 跨 skill 路由錯 | routing-error / scope-overlap |

**重要**：自動分類後務必跟使用者 confirm 一次。智能判斷會飄，confirm 是穩定 log 品質的關鍵。

---

## 診斷規則映射（給 skill-evolve 用）

| 規則 | 觸發條件 | 結論 |
|---|---|---|
| R1 | 同一 tag 累積 ≥ 3 條 | 結構性訊號，不是偶發 |
| R2 | 同一 skill 跨 ≥ 3 個 tag | 該 skill 整體該重構 |
| R3 | `coverage-gap` 描述同一主題 ≥ 2 次 | reference 該抽出獨立檔案 |
| R4 | `trigger-miss` 反覆出現 | description 觸發詞需擴充 |
| R5 | `trigger-overreach` 反覆出現 | description 需加 DO NOT trigger 邊界 |
| R6 | `intent-misread` 反覆出現 | SKILL.md 主流程需加 Step 0 意圖判讀 |
| R7 | `scope-overlap` 同一對 skill ≥ 2 次 | 兩個 skill 該重新切邊界或合併 |
| R8 | `routing-error` 同一個父 skill ≥ 2 次 | 父 skill 的協作邏輯要強化 |

**閾值補丁（時間跨度）**：R1 觸發後若 `span_days < 7`，降級為「需觀察」（避免單次事件誤判為趨勢）。

---

## 變更紀錄

| 日期 | 變更 | 原因 |
|---|---|---|
| 2026-04-26 | 初版 7 個 tag | 從 8 個原始 tag 經 MECE 檢驗收斂 |
| 2026-04-26 | 移植到 GitHub repo (claude-evolve) | claude.ai 環境不持久,改用 web_fetch 模式 |
