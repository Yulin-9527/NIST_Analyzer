# NIST V2.1 BIOS Verifier

![Version](https://img.shields.io/badge/version-1.1-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg)
![ML-DSA-87](https://img.shields.io/badge/PQC-ML--DSA--87-purple.svg)

離線檢查一整顆 SPI flash dump 是否通過 **NIST V2.1（D051 2.1）** 驗證的圖形化工具。
拖一個 `.bin` 進來就好 —— BIOS base、EC region、NVRSV（primary / backup）、公鑰位址
全部由 image 自己推導，不需要先填設定。

## 功能特色

- **自動位址推導**：Flash Descriptor → BIOS base / PDR，EC region 由 6 個連續 `0xA5`
  特徵定位，公鑰預設取 `EC 起點 + 0x30000`
- **完整驗證鏈**：NVRSV 結構 → manifest hash（SHA-256 / SHA-384 / SHA-512）→
  **ML-DSA-87 後量子簽章**（V2.0 image 自動改用 RSA PKCS#1 v1.5）→ NVD digest
- **PQC 不會靜默跳過**：找不到公鑰時簽章驗證判為 **FAIL「未執行（缺公鑰）」**，
  整體結果不會因為「沒驗」而變成 PASS
- **位址來源透明**：每一側標明 **DYNAMIC**（descriptor 推導且真的命中 `_NV_RSV_`）、
  **FALLBACK**（候選都沒中，改用掃描）或 **NONE**（該側沒有備份副本）
- **建議 firmware 旗標**：依 image 產出 `oem_project.h` 片段
  （`SUPPORT_INTEL_NVRSV_NVD_BLOCK`、`SUPPORT_NIST_NVRSV_FORCE_16M_OFFSET`、
  `SUPPORT_NIST_NVD*` …），換 BIOS 時直接整段貼回
- **原始資料檢視**：公鑰與簽章的前 / 後 256 bytes、每個 manifest 的起點 / size /
  stored hash / 實算 hash，逐 byte 差異標紅
- **報表輸出**：HTML（可折疊原始資料）與 JSON；另有 CLI 可批次跑
- **離線執行**：不連網、不寫入任何 flash

## 系統需求

- **作業系統**：Windows 10 / 11（64-bit）
- **記憶體**：建議 4GB 以上
- **儲存空間**：約 60MB

## 下載與安裝

前往 [Releases](https://github.com/Yulin-9527/NIST_Analyzer/releases) 下載最新版
`NIST_V21_Verifier.exe`。

本工具為免安裝版本：下載 → 放任意資料夾 → 雙擊執行。

> **注意**：首次執行時 Windows SmartScreen 可能出現警告，請點「更多資訊」→「仍要執行」。

## 使用說明

### 基本操作

1. 啟動 `NIST_V21_Verifier.exe`
2. 把 BIOS ROM（`.bin`）拖進視窗，或按 **瀏覽…** / `Ctrl+O`
3. 按 **開始驗證**（或 `F5`）

驗證中會顯示進度條；完成後上方出現結論 banner：

```
✓ PASS — 這顆 BIOS 可通過 NIST V2.1 驗證
✗ FAIL — PQC 簽章驗證：未執行（缺公鑰）
```

### 欄位說明

| 欄位 | 預設 | 說明 |
|---|---|---|
| **BIOS ROM** | — | 完整 flash dump（含 descriptor），不是 BIOS region only |
| **EC 起點** | auto | 留空 = 在 image 內找 6 個連續 `0xA5`，取所在 block 的 4KB 邊界 |
| **公鑰位址** | auto | 留空 = EC 起點 + `0x30000`（ML-DSA-87 公鑰 2592 B）|
| **PDR→backup** | auto | 留空 = 依序試 `0x180000` / `0x1E0000` / `0x200000` |
| **同時驗 backup** | 開 | 關掉就只驗 primary 側 |

公鑰來源：先讀 BIOS ROM 內的 EC（EC 起點 + `0x30000`）；該區是 `0xFF/0x00`
（dump 沒帶 EC 或 EC 沒燒 key）時，才自動改用同目錄的 EC 檔。

### 結果分頁

| 分頁 | 內容 |
|---|---|
| **驗證結果** | 最上面是「驗證摘要」（PQC / manifest / NVD），往下是分組明細；每列可雙擊複製、滑過看完整內容 |
| **建議 config** | 依 image 推導的 `oem_project.h` 旗標片段，可複製或匯出 |
| **原始資料 (JSON)** | 完整結構化結果 |
| **原始資料 (hex)** | 左邊選項目、右邊看 bytes；manifest 為 stored vs computed 逐 byte 對照 |

### 快捷鍵

| 快捷鍵 | 功能 |
|---|---|
| `Ctrl+O` | 開啟 BIOS ROM |
| `F5` | 開始驗證 |
| `Ctrl+H` / `Ctrl+J` | 匯出 HTML / JSON |
| `Ctrl+K` | 匯出建議旗標 |
| `F1` | 關於 |
| `Ctrl+Q` | 結束 |

## 判定規則

| 檢查 | 必要 |
|---|---|
| Flash Descriptor（FLVALSIG / FRBA / FLREG） | ✅ |
| BIOS base（`byte@0x45` ×1MB + `byte@0x44` ×4KB） | ✅ |
| NVRSV offset 命中 `_NV_RSV_`（固定 `0x20000` / size-aware 都試） | ✅ |
| NVRSV 結構與 V2.1 佈局（含 `_NVD`） | ✅ |
| 每個 manifest 的 hash 比對 | ✅ |
| **PQC 簽章（ML-DSA-87）或 V2.0 的 RSA 簽章** | ✅（缺公鑰 = FAIL）|
| backup 側 NVD digest（SHA-384） | ✅ |
| primary 側 NVD digest | 僅警告（NVD 是活資料，digest 從寫下就過期）|
| manifest 範圍與 NVD 重疊 | 警告（manifest 定義本身的問題）|

## 位址怎麼推導

| 項目 | 來源 |
|---|---|
| EC region | 6 個連續 `0xA5`，取所在 block 的 4KB 邊界（例：`0xA5` 在 `0x4040` → `0x4000`）；descriptor `dword@0x00` 僅作交叉檢查 |
| 公鑰 | EC 起點 + `0x30000`（2592 B） |
| BIOS base | `byte@0x45` × `0x100000` + `byte@0x44` × `0x1000`（與 `FLREG[1].base` 交叉驗證）|
| primary NVRSV | BIOS base + offset（`0x20000` 或 `(bios_size − 16MB) + 0x20000`，取命中者）|
| backup NVRSV | `FLREG[4].base` + `0x180000` / `0x1E0000` / `0x200000` + offset（命中才算 DYNAMIC）|

## 建議 firmware 旗標（範例）

```c
#define SUPPORT_NIST_V2                 TRUE
#define NIST_V2_SIG_SCHEME              NIST_V2_SIG_MLDSA87
#define SUPPORT_INTEL_NVRSV_DYNAMIC     1
#define SUPPORT_INTEL_NVRSV_NVD_BLOCK   1
#define SUPPORT_NIST_NVRSV_FORCE_16M_OFFSET  1   // 這顆 image 實際用 +0x20000
#define SUPPORT_NIST_NVD                1
#define SUPPORT_NIST_NVD_WDT            1
#define SUPPORT_NIST_NVD_RECOVERY       0
#define SUPPORT_NIST_TEST_DISPATCH      1
```

完整片段（含註解與其餘旗標）由 GUI 的 **建議 config** 分頁產生。

## 命令列

```bash
NIST_V21_Verifier.exe dump.bin              # 開 GUI 並驗證
python verify_cli.py dump.bin               # 批次，exit code 0=PASS 1=FAIL
python verify_cli.py *.bin --quiet          # 只印結論
python verify_cli.py dump.bin --html r.html --json r.json
python verify_cli.py dump.bin --extras      # 印出公鑰 / 簽章 / manifest 原始資料
python verify_cli.py dump.bin --config-out config.json
python verify_cli.py dump.bin --ec-base 0x4000 --pubkey-addr 0x34000
```

## 技術規格

### NVRSV 結構（V2.1）

| 位移 | 內容 |
|---|---|
| `+0x00` | `_NV_RSV_` 標記 |
| `+0x08` / `+0x0C` | 版本 / 結構大小 |
| `+0x10` / `+0x14` | BIOS version / SVN |
| `+0x18` | hash_count（manifest 數量）|
| `+0x40` | HashManifest[0]（每筆 `0x10` + hash 長度）|
| manifests 之後 | `_NVD` info（V2.1 才有，`0x10` bytes）|
| 之後 | 簽章 header `NVRS`（TBS / scheme / param）|
| 之後 | 簽章值（ML-DSA-87 = 4627 bytes）|

### 簽章

- **ML-DSA-87**：scheme `0x00A1`，公鑰 2592 B，簽章 4627 B（V2.1）
- **RSA**：PKCS#1 v1.5，RSA-2048 + SHA-256 / RSA-4096 + SHA-384（V2.0 image）

### NVD

`_NVD` 記錄資料位移、大小、hash algo 與 hash 存放位移（實測 `0xF000`）；
digest 只在 **backup** 側作為 gate。

## 常見問題

### Q1：為什麼顯示「不是有效的 flash dump」？
**A**：檔案裡找不到 Flash Descriptor（`FLVALSIG = 0x0FF0A55A` @`0x10`）。
請確認是**完整 flash dump**，而不是只截 BIOS region。

### Q2：PQC 顯示「未執行（缺公鑰）」怎麼辦？
**A**：BIOS ROM 內 `EC 起點 + 0x30000` 是空白的，同目錄也沒有 EC 檔。
把對應的 EC image（內含 2592 B 公鑰）放在 BIOS 同一個資料夾，或用
`--key EC_release.bin` 指定。**沒有金鑰就不能算通過。**

### Q3：backup 顯示 FALLBACK 是什麼意思？
**A**：`PDR + 0x180000 / 0x1E0000 / 0x200000` 三個候選都沒有 `_NV_RSV_`，
改用在 image 內掃到的第二個 `_NV_RSV_`。這代表這顆 BIOS 的 PDR 位移不是常見值，
位址仍然可用，但要知道它不是「算出來的」。

### Q4：某個 manifest 一直 FAIL？
**A**：先看「manifest 與 NVD 重疊」警告 —— manifest 的 hash 範圍如果吃到 NVRAM
（`_NVD`）區域，任何變數寫入都會讓該 hash 永遠不過。這是 manifest 定義本身的問題，
不是驗證程式。

### Q5：primary 的 NVD digest 為什麼是警告不是失敗？
**A**：primary 的 NVD 是活資料，digest 從寫下那一刻就過期。只有 backup 側
（快照）的 digest 才當作 recovery 前的 gate。

### Q6：V2.0 的 BIOS 可以驗嗎？
**A**：可以，但會顯示 `格式版本 V2（無 _NVD）` 並判 FAIL —— 這是 V2.1 驗證工具。
V2.0 的 x86 結構仍會照常解析、hash 與簽章也會驗，方便你分辨「結構不對」還是「簽章不對」。

### Q7：需要連網嗎？
**A**：不需要。全程離線，也不會寫入任何 flash。

## 版本歷程

### v1.1 (2026-11)

- 全新 V2.1 驗證核心：自動位址推導、PQC（ML-DSA-87）、NVD digest
- 位址來源標示 DYNAMIC / FALLBACK / NONE；backup 多常數自動 probe
- PQC 缺公鑰 = FAIL，新增「驗證摘要」群組
- 建議 firmware 旗標（`oem_project.h` 片段）與舊版 `config.json` 匯出
- 原始資料檢視（公鑰 / 簽章前後 256 B、manifest stored vs computed 逐 byte 對照）
- 四分頁介面、選單列與「關於」（`F1`）、CLI 擴充（`--extras` / `--ec-base` / `--pubkey-addr`）

### v1.0.6 (2026-02-04) — 舊版 `NIST_Analyzer`

- 新增 RSA-4096 簽名驗證（SHA-384）
- 新增略過 Header 模式（手動指定 NV_RSV / BIOS 起始位址）
- 修正 PDR 備份區塊解析邏輯

### v1.0.5 (2025-11-28) — 舊版 `NIST_Analyzer`

- 新增 PDR 備份區塊驗證（Backup NVRSV / IBB / WBIO）
- 設定檔 `config.json` 持久化
- 報告改為 HTML

### v1.0.1 (2025-11-10) — 舊版 `NIST_Analyzer`

- 首次發布：NIST BIOS Bin 解析、RSA-2048 驗證、圖形化介面

## 授權條款

本專案採用 MIT 授權條款 - 詳見 [LICENSE](LICENSE) 檔案

## 作者

**Yulin Hsieh**

- GitHub: [@Yulin-9527](https://github.com/Yulin-9527)

## 致謝

- 使用 [PyQt5](https://pypi.org/project/PyQt5/) 開發圖形介面
- 使用 [dilithium-py](https://github.com/GiacomoPope/dilithium-py) 實作 ML-DSA-87 驗證
- 使用 [PyCryptodome](https://www.pycryptodome.org/) 實現 V2.0 的 RSA 驗證

## 安全聲明

本工具僅用於合法的 BIOS 檔案分析與驗證用途。使用者應遵守所有適用的法律法規，
不得將本工具用於任何非法目的。

---

**⭐ 如果這個專案對您有幫助，請給我們一個星星！**

#### [喜歡的話歡迎給個打賞吧!](https://ganknow.com/yulin9527/tip)
#### 歡迎用 BTC 支援：
#### [bc1px0u84lk50jugxzwrcu26h2gsd00ef78pqplrr7sv96qgt8fgnrqqur0x7e](https://blockstream.info/address/bc1px0u84lk50jugxzwrcu26h2gsd00ef78pqplrr7sv96qgt8fgnrqqur0x7e)
#### 掃描 QR Code 進行 BTC 捐贈：
![BTC Donate QR Code](https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=bitcoin:bc1px0u84lk50jugxzwrcu26h2gsd00ef78pqplrr7sv96qgt8fgnrqqur0x7e)
