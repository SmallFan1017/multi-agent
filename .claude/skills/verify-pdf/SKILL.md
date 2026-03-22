---
name: verify-pdf
description: "驗證 PDF 完整性（header/EOF 檢查 + 大小），自動重新下載截斷或損壞的檔案，直到驗證通過為止。改造自官方 /loop 的「反覆執行直到條件達成」結構。"
---

# /verify-pdf — PDF 完整性驗證

驗證下載的 PDF 是否完整，自動重試直到全部通過。

## 使用方式

```
/verify-pdf                                  # 驗證 output/papers/ 下所有 PDF
/verify-pdf output/papers/icassp/            # 驗證指定資料夾
/verify-pdf output/papers/icassp/2401.08992_xxx.pdf  # 驗證單一檔案
```

## 執行流程

> 改造自官方 `/loop`：對每個問題檔案反覆重下載，直到驗證通過（最多 3 次）。

**Step 1 — 掃描目標**

列出所有待驗證的 PDF 路徑。

**Step 2 — 對每個 PDF 執行完整性檢查**

```python
import os

def verify_pdf(path):
    if not os.path.exists(path):
        return "missing"
    size = os.path.getsize(path)
    if size < 50 * 1024:          # < 50 KB
        return "too_small"
    with open(path, 'rb') as f:
        header = f.read(5)
        f.seek(-30, 2)
        tail = f.read()
    if header != b'%PDF-':
        return "bad_header"
    if b'%%EOF' not in tail:
        return "truncated"
    return "ok"
```

**Step 3 — 核對 arXiv 標題**

前往 `arxiv.org/abs/{arxiv_id}` 確認論文標題與檔名關鍵字一致。

**Step 4 — 自動重新下載（/loop 結構）**

若 Step 2 或 Step 3 發現問題，立即重新下載並重新驗證，最多重試 3 次：

```powershell
Invoke-WebRequest -Uri "https://arxiv.org/pdf/{ARXIV_ID}" `
    -OutFile "{目標路徑}" `
    -Headers @{"User-Agent"="Mozilla/5.0"} `
    -TimeoutSec 120
```

**Step 5 — 輸出驗證報告**

## 輸出格式

```markdown
## ✅ PDF 驗證報告

| 檔案名稱 | 大小 | %PDF- | %%EOF | 標題核對 | 狀態 |
|---------|------|-------|-------|---------|------|
| 2401.08992_efficient_adapter.pdf | 276 KB | ✅ | ✅ | ✅ 吻合 | ✅ 正常 |
| 2506.12154_whisper_two_pass.pdf  | 12 KB  | ✅ | ❌ | ✅ 吻合 | ⚠️ 截斷 → 重新下載 |
| 2506.12154_whisper_two_pass.pdf  | 1755 KB| ✅ | ✅ | ✅ 吻合 | ✅ 修復完成 |
```

若重試 3 次仍失敗，回報錯誤並跳過，不阻塞其他檔案的驗證。
