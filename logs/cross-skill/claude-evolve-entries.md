# claude-evolve log entries — 2026-04-27

> 兩條獨立的錯誤模式，分開記。
> Skill: strategy-case-report
> Version: v2.7.1 → v2.7.2
> Trigger: external review by user

---

## Entry 1｜大版本升級時，主檔改完但 references 殘留舊路徑詞

**Skill**: strategy-case-report
**Version**: v2.7.0 主檔升級（移除 PDF / Slide / HTML artifact / report-renderer 渲染路徑）
**Detected at**: v2.7.2（兩個小版本之後才被外部審查抓到）
**Severity**: medium（不會讓 skill 完全失靈，但會在執行中誤導 AI 走回舊路徑）

### 症狀

v2.7.0 把架構從「分析 → JSON checkpoint → render-renderer 渲染」改成「對話內 markdown 邊寫邊展示」。SKILL.md 主檔的描述、Phase 6、Hard Rules 都同步改了，但 references/ 子文件留下四處舊路徑詞：

1. `SKILL.md` 原則 1 列舉決策清單時仍寫「視覺決策」——v2.7 已不做視覺渲染，這個決策類型已不存在
2. `case-selection.md` 來源驗證規則仍寫「配圖使用時」「最終輸出附 image source appendix」——v2.7 不產出圖像
3. `critique-scorecard.md` 執行邏輯仍寫「全部 ✅ → 直接進截點輸出（或路徑 D 輸出）」——「截點輸出」「路徑 D」是 v2.6 以前的多輸出路徑用詞
4. `model-library.md` Rule 3 仍寫「所有被選中的模型，都要設計成可上簡報的圖表」——v2.7 不做簡報

### 為什麼會發生

v2.7.0 升級時做了以下事情：
- 改 SKILL.md frontmatter description
- 改 Phase 6 渲染流程
- 改 Hard Rules
- 改 Reference 讀取規則表格

但**沒有對 references/ 做全文掃描**，假設了「主檔改完，子文件會被視為被覆蓋」。實際上 references/ 在 SKILL.md 不變動的位置仍有大量上一代敘事文字，這些文字被讀進 context 時 AI 會混淆「現在到底走哪條路徑」。

### 正確做法

大版本升級（major path change，例如移除整條輸出路徑、新增整個 phase、改變決策結構）時，**必須執行 references/ 全文掃描**，搜尋所有路徑詞、模式名、版本標記，逐一確認是「應該保留的歷史脈絡」還是「應該刪除/改寫的殘留」。

最低操作：
```bash
cd <skill_dir>
grep -rn "<舊路徑詞 1>\|<舊路徑詞 2>\|..." SKILL.md references/
```

例如本案應掃描的詞包含：「視覺決策」「配圖」「image source」「截點輸出」「路徑 D」「上簡報的圖表」「PDF」「report-renderer」「Sprint Mode」「Full Mode」。

掃完一輪後逐項判斷：
- 完全是舊路徑語言 → 刪或改寫
- 是當前語言但用詞混淆 → 統一用詞
- 是歷史 changelog 必要保留 → 加註「v2.X 已移除」

### 為什麼這條值得記

主檔（SKILL.md）改動會被 review，因為它是 entry point；但 references/ 是 progressive disclosure 的下游，review 時容易被當作「主檔改了，子檔自然會配合」而漏掃。**這是 skill 大版本升級的系統性盲點，不是某個 skill 的個別問題**——任何進入 v2 → v3、v3 → v4 的 skill 都會踩到。

### 結構性教訓（給未來 skill 升級用）

- 主檔 description / 流程改了不等於 references 也改了
- references 的更新成本看起來低，所以容易被推遲到「下次再說」，然後就遺忘
- v2.7.0 升級到 v2.7.2 中間隔了至少兩個小版本才被抓到——意味著如果沒有外部審查，這些殘留可能永遠存在
- **建議在 skill-create / skill-review 流程內，把「references/ 全文掃描」設為大版本升級的必檢項**

---

## Entry 2｜「批判內化，不外顯」設計過於極端，容易讓批判實質消失

**Skill**: strategy-case-report
**Version**: v2.3 引入「批判內化，不外顯」→ v2.7.2 修正為「批判內化執行，最小外顯紀錄」
**Detected at**: v2.7.2 external review
**Severity**: high（這是設計層面的失衡，不是文字殘留）

### 症狀

v2.3 為了減少使用者中斷成本，把 Phase 6 的 critique scorecard 設計為「AI 自主執行 → 直接修正 → 不外顯給使用者」。

設計動機是對的——避免每個批判問題都打斷使用者要求確認。但執行中容易退化為：

- AI 聲稱「執行了批判」，但使用者看不到任何修正痕跡
- 批判流程跑完後分析內容什麼都沒改，但流程紀錄寫「全部 ✅」
- 「不外顯」變成「批判可有可無」的免責條款

Gotcha G3 在 v2.7.1 已點名這個風險（「批判內化變成批判消失」），但只要求「批判執行記錄寫入來源附錄」——這不是強制可驗證痕跡，AI 可以寫一段空泛的「已執行批判，無重大發現」就過關。

### 為什麼會發生

「不外顯」是一個二元設定（外顯 / 不外顯），實務上需要的是光譜——

- **完全外顯**：scorecard 整張表丟給使用者勾選 → 中斷成本太高，違反 v2.3 設計動機
- **完全不外顯**：使用者看不到任何證據 → 批判可被空轉
- **最小外顯**：scorecard 不外顯，但批判修正後產生的「修正紀錄」必須外顯 → 保留設計動機 + 保留可驗證性

v2.3 / v2.7.1 卡在二元設定的「不外顯」極端，缺少「最小外顯紀錄」這個中間態。

### 正確做法

凡是設計「AI 自主執行某流程而不打擾使用者」時，必須區分兩件事：

1. **執行細節**（每一個檢核問題、每一個 scorecard 項目）→ 不外顯
2. **執行結果**（修正了什麼、降級了什麼、邊界調整了什麼）→ 外顯，且有最低欄位要求

對 strategy-case-report v2.7.2 的具體做法：
- Scorecard 整張表仍不外顯
- 但批判修正後**必須**產出一段「批判修正摘要」寫入最終報告，含四個必填欄位：
  - 修正了哪些過度推論
  - 哪些 confirmed 被降級為 estimated
  - 哪些 unknown 被移到知識邊界
  - 哪些 What to steal 因邊界不明而降級為策略觀察
- 全空白不允許；如果 AI 寫「全部欄位都是無」必須回頭重新檢視批判是否真正執行

### 為什麼這條值得記

「為了減少中斷而不外顯」是 AI skill 設計常見動機，但「不外顯」與「不執行」對使用者來說在輸出上是無法區分的。設計者必須主動引入「最小外顯紀錄」作為可驗證痕跡，否則 AI 行為會自然滑向「聲稱做了但實際沒做」的失效模式。

這不是 strategy-case-report 獨有的問題。任何「AI 自主執行內化批判 / 內化檢核 / 內化複查」的設計，都需要設計可驗證痕跡——否則內化就是消失。

### 結構性教訓（給未來 skill 設計用）

- 「不打擾使用者」與「沒有可驗證痕跡」是兩件不同的事，不能混為一談
- 二元設定（外顯 / 不外顯）通常是設計過簡的訊號——找中間態
- **最小外顯紀錄原則**：執行細節可內化，但執行結果必須有最低欄位要求的外顯紀錄
- Gotcha 寫「不要讓批判消失」是不夠的，必須在 SKILL.md 規格層級寫死「修正後必須產出 X 段含 Y 個必填欄位」
- 適用範圍：批判流程、檢核流程、複查流程、自我修正流程——任何「AI 自主跑一輪然後直接修正」的設計

---

## Cross-entry 觀察

兩條 entries 雖然是不同問題，但有一個共同點：**都來自外部審查，不是 AI 自查**。

- Entry 1 是 v2.7.0 升級時 references 沒掃乾淨，三輪迭代都沒被自查抓到
- Entry 2 是 v2.3 引入後存在三年（v2.3 → v2.7.1）才被審查指出「不夠硬」

這指向一個更上層的觀察：**skill 升級後的自查機制偏弱**。skill-review 是被動觸發（使用者主動要求審查時才跑），skill-summary 只記錄使用中踩到的坑，沒有「升級後主動掃描舊版本殘留」的機制。

待 skill-evolve 累積夠多 entries 後，這可能會浮現為一個 meta-pattern：「大版本升級後，舊版本的設計痕跡會在 references / Gotchas / 條件描述中殘留，需要主動 audit」。
