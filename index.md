# Log 檔索引

> skill-evolve 第一步會 fetch 這份檔案,知道目前 repo 內有哪些 log 檔。
> 使用者每次「新建」log 檔時要更新這份索引(append 既有檔不用動)。

---

## 維護規則

- **新建 log 檔時**：在對應分類下加一行
- **append 既有 log 檔時**：**不用動**這份檔案
- **刪除 log 檔時**：移除對應行（罕見,通常是該錯誤已經修好且不會再發生）

---

## 索引格式

每行一筆：

```
- {skill-name} / {tag} — 自 YYYY-MM-DD 起累積
```

`自 YYYY-MM-DD 起累積` 是該檔案的首筆紀錄日期，幫 skill-evolve 快速判斷資料新鮮度。

---

## 現存 Log 檔

### Skill 內部錯誤

<!-- 此區塊由使用者手動維護。每新建一個 logs/{skill}/{tag}.md 就在此加一行。 -->

(尚無紀錄)

---

### 跨 Skill 衝突

- cross-skill / scope-overlap — placeholder（尚未有實際紀錄）

---

## 為什麼需要這份索引

`web_fetch` 只能抓單一 URL，沒辦法列目錄內容。skill-evolve 需要先知道「這個 repo 有哪些 log 檔」才能決定要 fetch 什麼。

替代方案是「使用者每次告訴 evolve 哪些檔案存在」，但這違反「skill 應該自包含」原則——維護一份 index.md 比每次手動列清單可持續。
