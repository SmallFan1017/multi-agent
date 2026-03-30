---
name: find-papers
description: "搜尋特定主題的學術文獻，平行查詢多個來源（arXiv、Semantic Scholar、ACL Anthology），產出分級文獻回顧報告。改造自官方 /batch 的平行子任務架構。"
---

# /find-papers — 學術文獻搜尋

針對指定主題，平行搜尋多個來源，彙整成分級文獻回顧報告。

## 使用方式

```
/find-papers streaming ASR                          # 搜尋指定主題
/find-papers fine-tuning Whisper --venue Interspeech  # 限定會議
/find-papers adapter ASR --since 2023 --count 15   # 指定時間與篇數
```

### 預設值
- 時間範圍：近 3 年
- 目標篇數：10–15 篇
- 目標會議：Interspeech、ICASSP（可覆寫）

## 執行流程

> 改造自官方 `/batch`：將搜尋任務拆解為多個來源，平行執行後彙整。

**Step 1 — 解析搜尋參數**

從使用者輸入中提取：關鍵字、會議限定、時間範圍、目標篇數。

**Step 2 — 平行查詢（/batch 結構）**

同時查詢以下來源：

| 來源 | 方法 |
|------|------|
| arXiv 關鍵字搜尋 | `arxiv.org/search/?searchtype=all&query=...&order=-announced_date_first` |
| Semantic Scholar API | `api.semanticscholar.org/graph/v1/paper/search?query=...&fields=title,year,venue,externalIds` |
| ACL Anthology | `aclanthology.org` 搜尋（NLP/語音相關） |
| ISCA Archive | `isca-archive.org`（Interspeech 論文） |

**Step 3 — 去重與篩選**

- 合併各來源結果，去除重複
- 依相關度、引用數、發表場所排序
- 分為「高度相關 / 中度相關 / 參考但非核心」三級

**Step 4 — 輸出報告**

寫入 `output/literature-search/{topic}_{date}.md`

## 輸出格式

```markdown
# 文獻搜尋報告：[搜尋主題]

**搜尋日期**：[YYYY-MM-DD]
**關鍵字**：[使用的關鍵字]
**搜尋範圍**：[時間、來源]

---

## 高度相關

### 1. [論文標題]
- **作者**：[作者]
- **發表於**：[會議/期刊, 年份]
- **連結**：[URL]
- **摘要**：[1–2 句中文摘要]
- **與你研究的關聯**：[為什麼重要]
- **建議閱讀優先級**：⭐⭐⭐

---

## 中度相關

### N. [論文標題]
- ...
- **建議閱讀優先級**：⭐⭐

---

## 參考但非核心

- ...（⭐）

---

## 建議閱讀順序

1. 先讀 [論文X] — 因為 [原因]
2. 再讀 [論文Y] — 因為 [原因]

## 相關關鍵字（可進一步搜尋）
- [關鍵字 1]
- [關鍵字 2]
```

## 規範

- 論文標題保留英文原文，說明用繁體中文
- 若某來源無法存取，標註「未能查詢」並繼續其他來源
- 無法確認的資訊明確標示「未確認」
