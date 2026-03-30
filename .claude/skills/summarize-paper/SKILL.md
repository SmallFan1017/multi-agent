---
name: summarize-paper
description: "整理論文重點。概覽模式：多篇同時產出小卡幫助決定優先順序；詳細模式：單篇依章節深入整理。讀取 output/papers/ 下的本地 PDF。"
---

# /summarize-paper — 論文重點整理

兩種模式：快速概覽多篇，或深入整理單篇。

## 使用方式

```
/summarize-paper                        # 概覽模式：掃描 output/papers/ 下所有 PDF
/summarize-paper 2401.08992             # 概覽模式：指定單篇
/summarize-paper 2401.08992 --detail    # 詳細模式：對指定篇深入整理
```

---

## 📌 概覽模式（預設）

適合：同時面對多篇，需要決定「先看哪幾篇」。

### 流程

> ⚠️ 效率規則：每批 3–5 篇，提取完立即撰寫小卡，不等全部完成再一起寫。

**Step 0** — 用以下指令確認 PDF 清單（直接 print，不存中間檔）：
```python
import pdfplumber, glob, os
pdfs = glob.glob('output/papers/**/*.pdf', recursive=True)
for p in pdfs:
    print(os.path.basename(p))
```
若有缺少的 PDF，停止並提示使用者先下載。

**Step 1** — 分批提取（每批 3–5 篇），對每篇執行：
```python
with pdfplumber.open(pdf_path) as pdf:
    text = ''.join(p.extract_text() or '' for p in pdf.pages)
front = text[:12000]   # Abstract + Intro + Method
back  = text[-2000:]   # Conclusion
print(f"=== {arxiv_id} ===\n{front}\n--- BACK ---\n{back}")
```

**Step 2** — 立即撰寫本批概覽小卡，累積至回覆。

**Step 3** — 重複直到全部完成，末尾加主題分類表。

### 概覽輸出格式

```markdown
## [1] 2401.08992 — [論文標題]

**發表於**：[會議, 年份]
**核心問題**：[1 句話：解決什麼問題]
**方法摘要**：[1–2 句話：提出什麼方法]
**主要貢獻**：
- [成果 1（含數字）]
- [成果 2]

**與你研究的關聯度**：高 / 中 / 低
**建議閱讀優先級**：⭐⭐⭐ / ⭐⭐ / ⭐
```

輸出至：`output/summaries/overview.md`

---

## 📚 詳細模式（`--detail`）

適合：已決定要深入閱讀的特定篇目。

### 流程

1. 確認 PDF 存在於 `output/papers/`，否則停止並提示下載
2. 用 pdfplumber 提取全文（print 到終端，不存暫存檔）
3. 識別所有章節標題與層級
4. 逐章節詳盡整理（字數不限，細節盡量保留）
5. 每章末尾加 🔑 關鍵要點
6. 末尾加「與你研究的關聯性分析」

### 詳細輸出格式

```markdown
# [論文標題]

**作者**：[作者列表]
**發表於**：[會議/期刊, 年份]

---

## 論文概覽
[2–3 句話描述核心貢獻]

---

## 1. [章節標題]
[詳細內容]

### 🔑 關鍵要點
- [要點 1]

---

## 與你研究的關聯性分析

- **直接相關的方法/技術**：
- **可能的延伸方向**：
- **值得參考的實驗設計**：
- **注意事項/限制**：
```

輸出至：`output/summaries/{arxiv_id}_summary.md`

---

## 術語與公式規則

- 術語第一次出現：「中文（English, 縮寫）」
- 行內公式：`$...$`；獨立公式：前後空一行，`$$` 包圍並換行
- 全文使用**繁體中文**
