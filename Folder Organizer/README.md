# 🎯 Hermes 下載資料夾智能整理 Skill（skill.md）

> **讓亂雜的下載資料夾變成有條理的知識庫** – 只要一鍵或定時執行，Hermes 就會像私人檔案管家一樣，用語義理解、圖片視覺、OCR+LLM 等多模態 AI 技術，把每一個檔案放到它「該在」的地方，**絕不刪除**，保留原始時間戳，並給你一份清晰的執行報告。

---

## ✨ 為什麼選擇 Hermes？

| 功能 | 說明 | 你獲得的好處 |
|------|------|--------------|
| **語義分類，不靠副檔名** | 圖片依照主體、動作、環境、風格、意圖等多維標籤；文件依照專案、用途、對象等業務語義 | 再也不用在「IMG_1234.jpg」中尋找那張「金毛在公園奔跑」的照片 |
| **最多 8 層目錄樹** | 從專案 → 文件類型 → 具體用途層層遞進，彈性擴充 | 深度結構讓你一眼就能定位所需資料，同時不會過度碎片化 |
| **智慧重複偵測** | 感知雜湊（圖片）+ SHA‑256（檔案）+ MinHash（近似文件） | 自動把重複檔案集中到 `_Duplicates/`，釋放空間卻不丟失任何資料 |
| **時間戳完全保留** | 使用 `copy2` + `utime` 確保 `mtime/atime` 不變 | 避免破壞備份、版本控管或審計需求 |
| **零風險刪除** | 未經使用者明確授權，**絕不刪除**任何檔案 | 資料安全 100%，只搬移不刪除 |
| **即時回報與建議** | 每次執行產出 Markdown 報告，含統計、Top 5 目錄、重複提醒、未分類待審、以及可選的優化建議 | 你一眼就知道下載資料夾的健康狀態，並得到實用的下一步行動 |
| **主 Agent 永遠 Standby** | 所有耗時工作交由 4 個 Sub-agent 並行執行，主 Agent 只負責調度與即時回應 | 與你聊天時不會卡頓，隨時可下達 `hermes cleanup now` 等指令 |
| **彈性排程與手動觸發** | 支援 cron‑like RRULE（預設每日 03:00）、手動指令、檔案量/大小閾值觸發 | 依你的使用習慣自動啟動，或隨時想整理就立即執行 |
| **可擴充的 Plugin 架構** | 新增分類模型、自訂動作（解壓縮、上傳 NAS、產生縮圖）等 | 隨需求成長，不被死板的規則限制 |

---

## 🚀 快速上手

### 1️⃣ 安裝依賴（一次性）

```bash
# 建議使用 conda 或 venv 隔離環境
conda create -n hermes python=3.11 -y
conda activate hermes

pip install -r requirements.txt   # 包含 torch, transformers, paddleocr, opencv-python, imagehash, pyyaml, rich 等
```

> **提示**：如果你有 NVIDIA GPU，請先安裝對應版本的 `torch`（例如 `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121`），以獲得圖片分析的加速。

### 2️⃣ 放置 Skill 檔案

```bash
mkdir -p ~/.config/hermes/skill
cp hermes_cleanup_skill.md ~/.config/hermes/skill/skill.md
```

### 3️⃣ 基本設定（可選）

編輯 `~/.config/hermes/skill/config/schema.yaml` 調整一級目錄名稱、偏好的分類深度或加入專案別名。

```yaml
# 範例：把公寓專案命名為「租屋」而非「Apartment_*」
project_prefix:
  - "Apt_"
  - "租屋"
  - "房屋"
```

### 4️⃣ 首次執行（乾跑）

```bash
hermes cleanup --dry-run
```

- 會掃描 `~/Downloads`（或你在環境變數 `HERMES_DOWNLOAD_DIR` 中指定的路徑）
- 不會實際搬移檔案，只會在 terminal 顯示預估的分類結果與報告預覽。

### 5️⃣ 正式執行

```bash
hermes cleanup          # 按既定排程（或立即）執行一次完整整理
hermes cleanup now      # 立即觸發一次性任務，無論排程時間如何
```

### 6️⃣ 查看報告

執行完成後，報告會自動寫入：

```
~/.config/hermes/logs/hermes_cleanup_<RUN_ID>.md
~/.config/hermes/reports/hermes_cleanup_<RUN_ID>.md
```

你也可以用以下指令直接在 terminal 開啟最新報告：

```bash
hermes report latest
```

---

## ⏰ 預設排程（可自行修改）

Skill 內建一個 systemd‑timer / launchd / cron 範例，預設每日 **03:00** 執行：

```bash
# 以 systemd 為例（Linux）
cp ~/.config/hermes/skill/systemd/hermes-cleanup.timer ~/.config/systemd/user/
cp ~/.config/hermes/skill/systemd/hermes-cleanup.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now hermes-cleanup.timer
```

若你想改時段或頻率，編輯 `.timer` 檔案中的 `OnCalendar=` 欄位，例如：

```
OnCalendar=*-*-* 02:30:00   # 每天 02:30
OnCalendar=Mon..Fri *-*-* 04:00:00   # 週一至週五 04:00
```

重載後即生效：

```bash
systemctl --user daemon-reload
systemctl --user restart hermes-cleanup.timer
```

---

## 📊 報告範例（Markdown）

> 每次執行都會產出類似下面的報告，讓你一目了然。

```markdown
# 🧹 Hermes 下載資料夾整理報告
**Run ID:** hermes_20250617_030000_a1b2c3  
**執行時間:** 2025-06-17 03:00:12 – 03:04:38 (4m 26s)  
**觸發方式:** 排程 (Daily 03:00)

## 📈 全域統計
| 指標 | 數值 |
|------|------|
| 掃描檔案總數 | 12,847 |
| 成功分類搬移 | 11,923 (92.8%) |
| 移入 Unclassified | 612 (4.8%) |
| 偵測重複群組 | 87 組 (共 1,312 檔) |
| 釋放空間(重複檔) | 3.2 GB |
| **下載資料夾總大小** | **18.7 GB** |

## 📂 Top 5 一級子目錄佔用
| 排名 | 目錄 | 大小 | 檔案數 | 佔比 |
|------|------|------|--------|------|
| 1 | `02_Documents/ByProject/Apartment_A` | 4.1 GB | 1,204 | 21.9% |
| 2 | `01_Images/Subjects/Animals/Dog` | 3.6 GB | 3,421 | 19.3% |
| 3 | `06_AI_Models/Checkpoints` | 2.8 GB | 47 | 15.0% |
| 4 | `04_Media/Video` | 2.2 GB | 89 | 11.8% |
| 5 | `03_Code_Config/Projects` | 1.9 GB | 8,932 | 10.2% |

## 📦 主要搬移動作摘要
| 來源 | 目的地 | 檔案數 | 備註 |
|------|--------|--------|------|
| `~/Downloads/IMG_*.jpg` | `01_Images/Subjects/Animals/Dog/Running/` | 312 | 信心度 0.91±0.04 |
| `~/Downloads/aptA_bill_*.pdf` | `02_Documents/ByProject/Apartment_A/02_Bills/` | 18 | OCR 信心度 0.97 |
| `~/Downloads/screenshot_*.png` | `02_Documents/ByType/Finance/` | 45 | 銀行轉帳截圖 |
| `~/Downloads/*.lora.safetensors` | `06_AI_Models/LoRA/` | 23 | 模型檔自動辨識 |

## ⚠️ 重複檔提醒 (前 10 組)
| 群組 | 檔案數 | 總大小 | 主檔保留位置 | 建議 |
|------|--------|--------|--------------|------|
| `dup_001` (dog_running_01.jpg) | 4 | 48 MB | `01_Images/Subjects/Animals/Dog/Running/dog_running_01.jpg` | 其餘 3 份已移入 `_Duplicates/` |
| `dup_017` (Apt_A_lease.pdf) | 3 | 12 MB | `02_Documents/ByProject/Apartment_A/01_Contracts/Apt_A_lease.pdf` | 確認是否為歷史版本 |

> 👉 **完整重複清單**：`duplicate_reports/dup_hermes_20250617_030000_a1b2c3.json`

## ❓ 待人工覆核 (Unclassified)
- `99_Unclassified/Images/`：47 張（信心度 < 0.65）
- `99_Unclassified/Documents/`：12 份（掃描件模糊、手寫單據）
- **建議**：下次有空時打開 `00_Inbox/_Review_Queue/` 逐一確認，或回覆 `hermes review` 讓我協助。

## 💡 Agent 建議 (Optional)
1. **Apartment_A** 文件已超過 4 GB，建議每季封存至冷備份（NAS/雲端），只保留近 6 個月在本機。
2. `06_AI_Models/Checkpoints` 單一資料夾 47 個檔案 2.8 GB，考慮依模型架構 (SD1.5 / SDXL / Flux) 再分層，方便管理。
3. 發現 3 個 `.env` 檔案散落在 `03_Code_Config/Snippets/`，建議集中至 `Dotfiles/` 並加入 `.gitignore` 管理。

*下次排程執行：2025-06-18 03:00:00*  
*如需立即執行，請輸入 `hermes cleanup now`*
```

---

## 🛡️ 安全與隱私保證

- **本地處理**：所有圖片視覺、OCR、LLM 推論均在你自己的機器上完成（除非你自行啟用外部模型服務）。
- **不上傳**：檔案內容、檔名、路徑 **不會** 被發送至任何第三方伺服器。
- **禁止刪除**：除非你輸入 `hermes confirm delete <run_id>` 並再次確認，Skill 不會執行任何 `rm`、`unlink` 或 `trash` 操作。
- **時間戳保留**：搬移前後的 `mtime`、`atime`、`ctime` 透過 `shutil.copy2` + `os.utime` 完整保留，適合備份、版本控管與審計需求。

---

## 🔧 進階自訂（可選）

| 需要 | 怎麼做 |
|------|--------|
| **新增一級目錄** | 編輯 `config/schema.yaml`，在 `top_level_dirs` 陣列中加入名稱，重新載入設定 `hermes reload config`。 |
| **自訂分類規則** | 在 `config/rules/` 放置 YAML 或 JSON 檔案，定義關鍵字 → 目錄映射，Skill 會在啟動時自動合併。 |
| **加入後處理動作**（例如解壓縮、上傳 NAS） | 建立 Plugin：`~/.config/hermes/plugins/my_action.py`，實作 `run(file_path: Path) -> None`，然後在 `config/plugins.yaml` 註冊。 |
| **改變通知方式** | 編輯 `config/notifier.yaml`，支援 `stdout`, `desktop_notify`, `telegram`, `email`, `slack` 等。 |
| **調整 GPU 批次大小** | 環境變數 `HERMES_IMAGE_BATCH_SIZE=64`（預設 32），依顯存自行調校。 |

---

## 🙋‍♂️ 常見問題（FAQ）

**Q：Skill 會不會誤將重要檔案放錯地方？**  
A：所有分類都伴隨信心度分數。低於 0.65 的檔案會被放入 `99_Unclassified/` 並標記待審，你可以隨時檢查並手動調整。

**Q：我有很多重複檔案，會自動刪除嗎？**  
A：不會。重複檔案只會被移到對應類別的 `_Duplicates/` 子目錄，原始檔案仍保留。若你真的想刪除，請使用 `hermes confirm delete <run_id>` 並再次確認。

**Q：可以只整理特定類型的檔案嗎？（例如只整理圖片）**  
A：可以。執行時加上過濾參數：`hermes cleanup --type image`、`hermes cleanup --type document` 等。詳細選項請參閱 `hermes cleanup --help`。

**Q：我的下載資料夾在其他磁碟分區（例如 D:\），能用嗎？**  
A：當然。設定環境變數 `HERMES_DOWNLOAD_DIR=D:\Downloads` 或在 `config/path.yaml` 中指定即可。

---

## 📜 授權

此 Skill 使用 **MIT License**，你自由複製、修改、分發，甚至將其納入自己的商業產品。詳見檔案首授權聲明。

---

## 🎉 開始使用吧！

只要一行指令，你的下載資料夾從「黑洞」變成「知識園區」：

```bash
hermes cleanup now
```

然後打開最新報告，看看你的檔案如何被智慧地歸類、重複被集中、空間被釋放，而你仍掌握每一個檔案的命運。

> **讓檔案管理不再是苦差事，讓 Hermes 成為你最貼心的數據管家。** 🚀

--- 

*如果你在使用過程中有任何建議或發現 bug，歡迎至 GitHub Issue 或直接對我說 `hermes feedback <您的訊息>`，我會持續優化讓這個 Skill 越來越貼合你的工作流。*
