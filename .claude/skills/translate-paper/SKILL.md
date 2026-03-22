---
name: translate-paper
description: "將英文論文 PDF 翻譯為繁體中文，產出中英段落交錯對照的 Markdown 檔案。讀取 output/papers/ 下的本地 PDF，輸出至 output/translations/。"
---

# /translate-paper — 論文翻譯

將英文學術論文翻譯為繁體中文，採中英段落交錯對照格式。

## 使用方式

```
/translate-paper                                   # 列出 output/papers/ 下的 PDF 供選擇
/translate-paper output/papers/icassp/2401.08992_xxx.pdf   # 直接指定 PDF
```

## 執行流程

**Step 1 — 確認 PDF**
- 掃描 `output/papers/**/*.pdf`，確認目標 PDF 存在
- 若不存在：停止並提示使用者先透過 `/find-papers` 或 `@web-searcher` 下載

**Step 2 — 提取基本資訊**
列出作者、日期、發表於、arXiv ID

**Step 3 — 逐章節翻譯**
依原文順序翻譯，注意跨頁段落應依語意歸回正確章節

**Step 4 — 寫入輸出檔**
寫入 `output/translations/{arxiv_id}_translation.md`

## 中英對照格式

每個段落先呈現英文原文（引用區塊），再接中文翻譯：

```markdown
> **[原文]** We propose a novel approach...
>
> **[翻譯]** 我們提出一種新穎的方法...
```

## 術語規則

- 第一次出現：「中文（English, 縮寫）」，例如「自動語音辨識（Automatic Speech Recognition, ASR）」
- 後續可直接使用中文或縮寫

## 數學公式規則

- 行內：`$...$`
- 獨立公式：前後空一行，`$$` 後立即換行，結尾 `$$` 前換行
- 有編號時用 `\tag{n}` 寫在 `$$` 內部

```
$$
\text{Loss} = -\sum_{t} \log p(y_t | x) \tag{1}
$$
```

## 完成後自我檢查

1. 每頁文字均已翻譯，無段落遺漏
2. 段落數與原文一致
3. 數學公式 LaTeX 語法正確
4. 術語第一次出現已標註英文
