# Workspace Standards

這份文件定義本工作區（研究生 AI Agent 系統）的**通用規範**；主入口檔請參考 `AGENTS.md` / `CLAUDE.md`。

## 通用輸出

- **語言**：預設使用繁體中文。
- **術語**：第一次出現採「中文（English, 縮寫）」；後續可用中文或縮寫。
- **格式**：主要輸出為 Markdown。

## LaTeX（HackMD / VS Code 相容）

- 行內公式：`$...$`
- 獨立公式：前後各空一行，並使用

$$
...\tag{1}
$$

- 若有公式編號，必須寫在 `$$ ... $$` 內部，用 `\\tag{}`。
- 工具寫檔注意：若內容需放進 **JSON 字串**（例如透過工具呼叫傳遞整段 Markdown），`
` 內的反斜線需做 JSON escaping（例如 `\text` 在 JSON 內要寫成 `\\text`），避免寫入後變成 `text`。

## PDF 優先原則（重要）

- 翻譯與摘要**以本地 PDF 為準**：優先從 `output/papers/` 讀取，不改抓 arXiv HTML。
- 若 PDF 不存在，應先停止並提示使用者透過 `@web-searcher` 下載後再繼續。

## 效率與中間產物

- 除非使用者要求，避免建立中間暫存檔（例如 `_extract.json`、`_content.txt`）。
- 多篇處理建議「每批 3–5 篇」：提取 → 立即處理 → 立即寫入，避免 token 超量。

## 建議的預設輸出位置（可被使用者覆蓋）

- 翻譯：`output/translations/{arxiv_id}_translation.md`
- 概覽摘要（多篇）：`output/summaries/overview.md`
- 詳細摘要（單篇）：`output/summaries/{arxiv_id}_summary.md`

## 調度回報格式

任務完成後需回報：

```text
## 📋 調度報告
- **任務**：[使用者的原始需求]
- **調度的 Sub-Agent**：
  1. @paper-translator — [做了什麼]
  2. @paper-summarizer — [做了什麼]
- **產出檔案**：[列出產生的檔案路徑]
- **備註**：[任何需要使用者注意的事項]
```
