# GCP ESPnet 語音辨識專案規劃

## 專案目標

以最小可行方式跑一個 ESPnet2 語音辨識（ASR）推論服務，打包成 Docker 映像，部署至 GCP VM 上執行。

---

## 專案目錄結構

```
gcp_espnet/
├── README.md               # 本規劃文件
├── Dockerfile              # Docker 映像建置腳本
├── .dockerignore           # Docker 建置排除清單
├── requirements.txt        # Python 依賴套件
├── Makefile                # 常用指令快捷鍵
├── src/
│   └── asr_inference.py    # 最小 ESPnet2 ASR 推論腳本
├── scripts/
│   ├── download_model.sh   # 預訓練模型下載腳本
│   └── setup_gcp.sh        # GCP 環境設置腳本（建立/管理 VM）
├── audio_samples/          # 測試用音訊檔（.wav 16kHz mono）
│   └── .gitkeep
└── configs/
    └── inference.yaml      # 推論參數設定
```

---

## 技術選型

| 元件 | 選擇 | 理由 |
|---|---|---|
| ASR 框架 | ESPnet2 (`espnet_model_zoo`) | 最小 API、預訓練模型易取得 |
| 預訓練模型 | `pyf98/librispeech_100_conformer_ctc` (HuggingFace Hub) | 輕量、英文基準測試佳 |
| 容器化 | Docker (python:3.10-slim 為基底) | 輕量、GCP 相容 |
| 程式碼倉庫 | GitHub | 版本管理、VM 直接 clone 部署 |
| 運算平台 | GCP Compute Engine (e2-standard-4) | 4 vCPU / 16GB，足夠 CPU 推論 |

> **GPU 版本**：若需加速，可改用 `n1-standard-4 + T4` GPU，並替換為 CUDA 基底映像。

---

## 實作階段規劃

### Phase 1 — 本地端最小 ESPnet 推論範例

**目標**：能在本機執行 `python src/asr_inference.py audio_samples/test.wav`，輸出轉錄文字。

**步驟**：
1. 建立 Python 虛擬環境
   ```bash
   python3 -m venv venv && source venv/bin/activate
   pip install -r requirements.txt
   ```
2. 下載預訓練模型（首次執行自動快取至 `~/.cache/espnet`）
3. 執行推論腳本

**驗收標準**：輸入 WAV 檔，終端機輸出辨識文字，無錯誤。

---

### Phase 2 — Docker 化

**目標**：`docker build` 後，`docker run` 可直接執行 ASR 推論。

**步驟**：
1. 撰寫 `Dockerfile`（多階段建置，先安裝依賴，再複製程式碼）
2. 撰寫 `.dockerignore` 排除不必要檔案
3. 本地建置並測試：
   ```bash
   make build
   make test
   ```

**驗收標準**：
```bash
docker run --rm -v $(pwd)/audio_samples:/audio gcp-espnet:latest /audio/test.wav
# → 終端機輸出辨識文字
```

---

### Phase 3 — 推送至 GitHub

**目標**：將程式碼推送至 GitHub，GCP VM 直接 clone 並在 VM 上建置映像。

**步驟**：
1. 初始化 Git repository（若尚未建立）：
   ```bash
   git init
   git remote add origin https://github.com/<your-username>/gcp-espnet.git
   ```
2. 提交並推送：
   ```bash
   git add .
   git commit -m "feat: initial ESPnet ASR project"
   git push -u origin main
   ```

**驗收標準**：程式碼已在 GitHub repository 上可見。

---

### Phase 4 — GCP VM 部署與執行

**目標**：在 GCP Compute Engine VM 上 clone 程式碼、建置 Docker 映像並執行 ASR 推論。

**步驟**：
1. 建立 VM（Debian + Docker）：
   ```bash
   bash scripts/setup_gcp.sh create-vm
   ```
2. SSH 進入 VM：
   ```bash
   gcloud compute ssh espnet-vm --zone=asia-east1-b
   ```
3. 在 VM 上 clone 程式碼並建置映像：
   ```bash
   git clone https://github.com/<your-username>/gcp-espnet.git
   cd gcp-espnet
   docker build -t gcp-espnet:latest .
   ```
4. 上傳測試音訊並執行推論：
   ```bash
   # 另開終端機，從本地上傳音訊
   gcloud compute scp audio_samples/test.wav espnet-vm:/tmp/test.wav --zone=asia-east1-b

   # 在 VM 上執行
   docker run --rm \
     -v /tmp:/audio \
     -v espnet-model-cache:/root/.cache/espnet \
     gcp-espnet:latest /audio/test.wav
   ```

**驗收標準**：VM 終端機輸出辨識文字，無錯誤。

---

## 依賴套件說明（requirements.txt）

```
espnet==202402          # ESPnet2 核心
espnet_model_zoo        # 預訓練模型管理
torch==2.1.0+cpu        # PyTorch CPU 版（減少映像大小）
torchaudio==2.1.0+cpu
soundfile               # 音訊讀取
```

> **映像大小預估**：CPU 版約 3-4 GB（含模型快取則更大，建議模型掛載 Volume 而非烘入映像）

---

## 重要設計決策

### 模型快取策略（關鍵）

**問題**：ESPnet 模型約 300MB-1GB，不應每次容器啟動都重新下載。

**解法（選一）**：

| 方案 | 做法 | 適合情境 |
|---|---|---|
| A. Volume 掛載 | 下載一次掛載至 `/root/.cache/espnet` | 單一 VM、快速測試 |
| B. 預先烘入映像 | Dockerfile 內執行下載腳本 | 固定模型、生產部署 |
| C. GCS Bucket | 模型存放在 GCS，啟動時下載 | 多 VM、彈性擴展 |

本專案預設採用**方案 A**（開發階段），可依需求升級至方案 B/C。

---

## 執行流程總覽

```
本地端開發
    │
    ├─ [Phase 1] python src/asr_inference.py test.wav
    │               ↓ 輸出辨識文字 ✓
    │
    ├─ [Phase 2] docker build → docker run
    │               ↓ 容器內輸出辨識文字 ✓
    │
    ├─ [Phase 3] git push → GitHub
    │               ↓ 程式碼已上傳 ✓
    │
    └─ [Phase 4] GCP VM → git clone → docker build → docker run
                    ↓ 雲端輸出辨識文字 ✓
```

---

## 預估 GCP 費用（參考）

| 資源 | 規格 | 費用（約）|
|---|---|---|
| Compute Engine | e2-standard-4，asia-east1 | ~$0.134/小時 |
| 網路輸出 | SSH / SCP 傳輸音訊 | 幾乎免費 |

> 測試完畢後記得 **停止 VM** 以避免持續計費。

---

## 快速開始（本地端）

```bash
# 1. 建立虛擬環境
python3 -m venv venv && source venv/bin/activate

# 2. 安裝依賴
pip install -r requirements.txt

# 3. 準備測試音訊（需為 16kHz mono WAV）
cp your_audio.wav audio_samples/test.wav

# 4. 執行推論
python src/asr_inference.py audio_samples/test.wav

# 5. 建置 Docker
make build

# 6. 測試 Docker
make test AUDIO=audio_samples/test.wav
```

---

## 快速開始（VM 端）

```bash
# 1. SSH 進入 VM
gcloud compute ssh espnet-vm --zone=asia-east1-b

# 2. Clone 專案
git clone https://github.com/<your-username>/gcp-espnet.git
cd gcp-espnet

# 3. 建置 Docker 映像（首次需時較長）
docker build -t gcp-espnet:latest .

# 4. 上傳音訊（另開本地終端機執行）
gcloud compute scp audio_samples/test.wav espnet-vm:/tmp/test.wav --zone=asia-east1-b

# 5. 執行推論
docker run --rm \
  -v /tmp:/audio \
  -v espnet-model-cache:/root/.cache/espnet \
  gcp-espnet:latest /audio/test.wav
```

---

## 後續擴展方向

- [ ] 加入 REST API（FastAPI）作為推論服務端點
- [ ] 換用 GPU 機型提升推論速度
- [ ] 整合 GCP Cloud Run（Serverless 容器執行）
- [ ] 支援中文 ASR 模型（ESPnet AISHELL 預訓練模型）
- [ ] 加入批次推論（多檔案並行處理）
