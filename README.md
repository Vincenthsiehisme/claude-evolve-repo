# claude-evolve

> Claude skill 進化系統的累積錯誤紀錄倉。
> 配合 [skill-summary](https://github.com/Vincenthsiehisme/...) 跟 [skill-evolve](https://github.com/Vincenthsiehisme/...) 兩支 Claude.ai 自訂 skill 一起用。

---

## 這是什麼

一個給 Claude.ai 環境的 **skill 自我進化基礎設施**。

Claude.ai 的對話之間沒有持久記憶，但 skill 用久了會踩重複的坑。這個 repo 就是把「踩坑紀錄」外部化到 GitHub，讓兩支 skill 跨對話協作：

- **skill-summary**（寫端）：每次踩坑判斷後產出 log 條目給我貼上來
- **skill-evolve**（讀端）：用 `web_fetch` 抓 raw URL，讀累積 log 做趨勢診斷

---

## 為什麼 public

log 內容只有：
- skill 名稱（例如 `erp-schema`、`task-dispatcher`）
- 錯誤模式描述（症狀 / 根因 / 優化方向）
- 觸發的診斷規則

**不含**：業務資料、客戶資訊、內部系統細節、原始對話內容。

Public 讓 `web_fetch` 可以直接抓 raw URL，不用 token 認證，架構最單純。

---

## 結構

```
claude-evolve/
├── README.md           ← 你正在看的這份
├── tags.md             ← 7 個 tag 的單一真實來源(skill-evolve 會 fetch)
├── index.md            ← 現存 log 檔索引(skill-evolve 第一步會 fetch)
└── logs/
    ├── {skill-name}/
    │   ├── {tag}.md
    │   └── ...
    └── cross-skill/
        └── scope-overlap.md
```

---

## 維護流程（給我自己看）

### 寫入流程（踩坑後）

1. 在 claude.ai 對話中跟 Claude 描述剛才踩的坑
2. 觸發 `skill-summary`，跑完 Step 0.5 智能判斷 + tag 分類 + confirm
3. summary 會給我三樣東西：
   - 格式化好的 log 條目（複製）
   - GitHub deep link（點擊跳到編輯頁）
   - 是否要更新 `index.md` 的提示（首次建立新檔案才需要）
4. 點 deep link、貼上 log 條目、commit

### 讀取流程（想跑診斷時）

1. 在 claude.ai 對話中觸發 `skill-evolve`
2. 告訴它「我的 repo 是 https://github.com/Vincenthsiehisme/claude-evolve」（首次需要,後續 evolve 可從 user memory 記住）
3. evolve 會：
   - fetch `tags.md` 確認 tag 集合
   - fetch `index.md` 列出現存 log 檔
   - 依 Step 0 情境逐一 fetch 需要的 log 檔
   - 套診斷規則，輸出報告 + 建議

---

## 寫入規範速查

完整規範見 `tags.md`。簡版：

| 內容 | 路徑 |
|---|---|
| 單 skill 內錯誤 | `logs/{skill}/{tag}.md` |
| 跨 skill 衝突 | `logs/cross-skill/scope-overlap.md` |

每筆紀錄格式（5 行內）：

```markdown
## YYYY-MM-DD — <一句話標題>

**Symptom**：<具體事實>
**Root cause**：<推測的根因>
**Suggested fix**：<優化方向>
**Skills involved**：<僅 cross-skill 用,字母排序>
```

新建檔案時加 H1 標題：

```markdown
# {skill-name} — {tag} 錯誤累積
```

---

## index.md 更新規則

**只在「新建 log 檔」時更新**，既有檔案 append 紀錄不用動 `index.md`。

更新方式由 skill-summary 在交付物中提示。
