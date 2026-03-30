# 📖 研究生 AI Agent 使用指南

## 一、系統架構

```
multi-agent/
├── AGENTS.md                              ← 主 Agent（VS Code Copilot 入口）
├── CLAUDE.md                              ← 主 Agent（Claude CLI 入口）
├── agents/
│   ├── paper-translator/AGENTS.md         ← 論文翻譯（中英對照）
│   ├── paper-summarizer/AGENTS.md         ← 論文重點整理
│   ├── code-reviewer/AGENTS.md            ← 程式碼審查
│   ├── presentation-maker/AGENTS.md       ← 簡報製作
│   ├── literature-search/AGENTS.md        ← 文獻搜尋
│   └── data-analysis/AGENTS.md            ← 數據分析
├── .claude/
│   └── skills/                            ← Claude CLI Skills（直接呼叫入口）
│       ├── find-papers/SKILL.md
│       ├── review-code/SKILL.md
│       ├── summarize-paper/SKILL.md
│       ├── translate-paper/SKILL.md
│       └── verify-pdf/SKILL.md
├── docs/
│   ├── usage-guide.md                     ← 本文件
│   └── workspace-standards.md             ← 工作區通用規範（單一真相來源）

```

---

## 二、如何使用

### 方式 A：VS Code Copilot（Agent Mode）

#### 前置條件
- VS Code 已安裝 GitHub Copilot 擴充功能
- 已登入 GitHub Copilot

#### 使用步驟

1. **開啟此工作區**：在 VS Code 中開啟 `multi-agent/` 資料夾
2. **開啟 Copilot Chat**：按 `Ctrl+Shift+I` 或點擊側邊欄的 Copilot 圖示
3. **切換到 Agent Mode**：在 Chat 面板頂部，切換模式為 **Agent**
4. **使用主 Agent**：直接輸入任務描述
   ```
   幫我翻譯這篇論文：C:\path\to\paper.pdf
   ```
5. **直接使用 Sub-Agent**：用 `@` 指定
   ```
   @paper-translator 翻譯這篇論文：C:\path\to\paper.pdf
   輸出到：C:\path\to\output\
   ```

#### 可用的 Agent 名稱

| 輸入 | 功能 |
|------|------|
| `@researcher` | 主 Agent，自動判斷並調度 |
| `@paper-translator` | 論文翻譯（中英對照） |
| `@paper-summarizer` | 論文重點整理 |
| `@code-reviewer` | 程式碼審查 |
| `@presentation-maker` | 簡報製作 |
| `@literature-search` | 文獻搜尋 |
| `@data-analysis` | 數據分析 |

---

### 方式 B：Claude CLI

#### 前置條件
- 已安裝 Claude CLI（`npm install -g @anthropic-ai/claude-code`）
- 已設定 API key

#### 使用步驟

1. **在終端機進入此工作區**：
   ```powershell
   cd C:\path\to\multi-agent
   ```

2. **啟動 Claude**：
   ```powershell
   claude
   ```
   Claude 會自動讀取 `CLAUDE.md` 作為系統指令。

3. **輸入任務**：
   ```
   幫我翻譯這篇論文：C:\path\to\paper.pdf
   輸出到：C:\path\to\output\
   ```

4. Claude 會自動：
   - 讀取對應的 Sub-Agent prompt（例如 `agents/paper-translator/AGENTS.md`）
   - 使用 Task tool 派出 Sub-Agent 執行
   - 完成後回報調度結果

---

## 三、常見使用範例

### 範例 1：翻譯一篇論文

```
幫我翻譯這篇論文，輸出中英對照格式
論文：C:\path\to\課程相關\some_paper.pdf
輸出到：C:\path\to\課程相關\翻譯\
```

### 範例 2：整理論文重點

```
幫我整理這篇論文的重點
論文：C:\path\to\課程相關\some_paper.pdf
輸出到：C:\path\to\課程相關\論文筆記\
```

### 範例 3：翻譯 + 整理（複合任務）

```
幫我處理這篇論文：翻譯並整理重點
論文：C:\path\to\課程相關\some_paper.pdf
輸出到：C:\path\to\課程相關\output\
```

### 範例 4：審查程式碼

```
我需要一個 Python 函式，用 sliding window 的方式處理串流音訊特徵。
以下是 AI 幫我寫的程式碼，請審查是否符合需求：

[貼上程式碼]
```

### 範例 5：製作簡報

```
幫我把這篇論文做成 lab meeting 的簡報，大約 15 分鐘
論文：C:\path\to\課程相關\some_paper.pdf
```

### 範例 6：搜尋文獻

```
幫我搜尋 Streaming ASR 相關的最新論文，特別是用 Transformer 架構的
時間範圍：2024-2026
```

---

## 六、快速驗收用 Prompts（建議先跑這些）

### 1) 單篇：翻譯（中英對照）

```
@paper-translator
翻譯論文（中英對照）。
論文：output/papers/interspeech/<你的檔名>.pdf
輸出到：output/translations/
```

### 2) 單篇：依章節詳細整理

```
@paper-summarizer
請用「詳細模式」依章節整理這篇論文。
論文：output/papers/interspeech/<你的檔名>.pdf
輸出到：output/summaries/
```

### 3) 複合任務：翻譯 + 整理（主 Agent 自動調度）

```
@researcher
幫我處理這篇論文：先翻譯（中英對照）再依章節整理重點。
論文：output/papers/interspeech/<你的檔名>.pdf
輸出到：output/
```

### 4) 驗證 AI 程式碼是否符合需求（你最常遇到的痛點）

```
@code-reviewer
我需要一個 Python 函式：輸入串流特徵 frames（shape: [T, D]），用 sliding window 做線上處理；要求：延遲 <= 320ms、支援 chunk overlap。
請審查下面程式碼是否符合需求，並指出會踩雷的 edge cases。

[貼上程式碼]
```

---

## 四、自訂與擴展

### 新增 Sub-Agent

1. 在 `agents/` 下建立新資料夾，例如 `agents/my-new-agent/`
2. 在裡面建立 `AGENTS.md`，格式參考現有的 Sub-Agent
3. 在根目錄的 `AGENTS.md` 和 `CLAUDE.md` 中加入新 Agent 的資訊

### 修改現有 Agent

直接編輯對應的 `AGENTS.md` 檔案即可，修改會在下次對話時生效。

> ⚠️ **同步提醒**：若修改的是根目錄的主 Agent（`AGENTS.md`），請同時修改 `CLAUDE.md`，兩個檔案的「使用者背景」、「調度規則」、「通用規範」必須保持一致。Sub-Agent（`agents/` 底下的檔案）只有一份，不需重複修改。

### 在其他目錄下使用 Agent

預設情況下，Agent 只在 `multi-agent/` 資料夾內有效。以下是在其他專案目錄使用的方法：

#### VS Code Copilot — 加入多根工作區（Multi-root Workspace）

最簡單的方式：將 `multi-agent/` 資料夾加入你目前的 VS Code 工作區。

1. 在 VS Code 選單：**檔案 → 將資料夾加入工作區**
2. 選擇 `C:\path\to\multi-agent`
3. 存成 `.code-workspace` 檔案（例如放在你的研究主目錄）

之後只要開啟該 `.code-workspace` 檔案，不論你在哪個專案，所有 `@researcher`、`@paper-translator` 等 Agent 都可以使用。

#### Claude CLI — 設定使用者層級 CLAUDE.md

Claude CLI 會自動往上層目錄尋找 `CLAUDE.md`，直到使用者根目錄。
可以在 `C:\Users\YourName\` 建立一個轉接檔，讓所有子目錄都能使用此 Agent：

```markdown
<!-- C:\Users\YourName\CLAUDE.md -->
# 全域研究生 Agent
請讀取並遵循以下檔案的指令：
C:\path\to\multi-agent\CLAUDE.md

Sub-Agent Prompt 的絕對路徑：
- 論文翻譯：C:\path\to\multi-agent\agents\paper-translator\AGENTS.md
- 論文整理：C:\path\to\multi-agent\agents\paper-summarizer\AGENTS.md
- 程式碼審查：C:\path\to\multi-agent\agents\code-reviewer\AGENTS.md
- 簡報製作：C:\path\to\multi-agent\agents\presentation-maker\AGENTS.md
- 文獻搜尋：C:\path\to\multi-agent\agents\literature-search\AGENTS.md
- 數據分析：C:\path\to\multi-agent\agents\data-analysis\AGENTS.md
```

這樣在 `C:\Users\YourName\` 下的任何子目錄執行 `claude`，都能自動載入此 Agent。

---

### 加入 MCP Server（選用，進階）

如果之後需要讓 Agent 存取外部工具（如網頁搜尋），可以設定 MCP Server：

#### VS Code 設定
在 VS Code 的 `settings.json` 中加入：
```json
{
  "mcp": {
    "servers": {
      "fetch": {
        "command": "npx",
        "args": ["-y", "@anthropic-ai/mcp-fetch"]
      }
    }
  }
}
```

#### Claude CLI 設定
在工作區建立 `.mcp.json`：
```json
{
  "mcpServers": {
    "fetch": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-fetch"]
    }
  }
}
```

> **前置需求**：安裝 Node.js（https://nodejs.org/）

---

## 五、FAQ

**Q：Agent 會自動讀取 PDF 嗎？**
A：在 VS Code Copilot Agent Mode 中，可以直接將 PDF 檔案拖拉到 Chat 中，或提供路徑讓 Agent 讀取。Claude CLI 也支援讀取本地檔案。

**Q：可以同時處理多篇論文嗎？**
A：可以，但建議一次處理一篇以確保品質。如果需要批次處理，告訴主 Agent 你有多篇論文，它會依序處理。

**Q：Agent 的回答品質不好怎麼辦？**
A：可以直接修改對應 Sub-Agent 的 `AGENTS.md` 來調整指令。常見調整：增加具體的格式要求、補充領域知識、修改輸出範例。

**Q：如何讓 Agent 記住我的偏好？**
A：你的基本偏好已寫在主 Agent 的「使用者背景」區塊。如果有新的偏好，直接編輯 `AGENTS.md` 或 `CLAUDE.md` 即可。

**Q：修改主 Agent 設定後只改了 AGENTS.md，有問題嗎？**
A：有問題。`AGENTS.md`（VS Code Copilot）和 `CLAUDE.md`（Claude CLI）是兩個獨立的入口，必須手動保持同步。記住：**改一個就要改另一個**。Sub-Agent（`agents/` 底下的檔案）只有一份，不受此影響。

**Q：不在 Agent/ 目錄下可以使用這個 Agent 嗎？**
A：可以，但需要設定。VS Code 請用「多根工作區」把 Agent/ 加進任何專案；Claude CLI 請在 `C:\Users\YourName\` 建立一個轉接 CLAUDE.md（詳見四、自訂與擴展）。
