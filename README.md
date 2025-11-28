# NIST Bin 分析工具

![Version](https://img.shields.io/badge/version-1.0.5-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey.svg)

一款簡易的 NIST BIOS Bin 檔案分析與驗證工具，提供圖形化介面，支援 RSA-2048 數位簽名驗證功能。

## 功能特色

- **完整結構解析**: 自動解析 NIST bin 檔案結構，包含 Header、NV_RSV、IBB、WBIO 等區塊
- **PDR 備份區塊驗證**: 支援 PDR (Platform Data Region) 備份區域驗證，包含 Backup NVRSV、IBB、WBIO
- **RSA-2048 簽名驗證**: 驗證各區域的數位簽名完整性
- **彈性化設定系統**: 可自訂公鑰位址、PDR 基底位址，設定持久化保存
- **直覺化圖形介面**: 基於 PyQt6 的現代化深色主題介面
- **詳細分析報告**: 生成包含十六進位數據、區塊分布、驗證結果的完整 HTML 報告
- **拖放操作**: 支援直接拖曳 bin 檔案到視窗進行分析
- **書籤導航**: 報告內建快速跳轉功能，方便瀏覽大型檔案
- **彩色狀態顯示**: 綠色/紅色標示驗證通過/失敗狀態

## 系統需求

- **作業系統**: Windows 10/11 (64-bit)
- **記憶體**: 建議 4GB 以上
- **儲存空間**: 50MB 可用空間

## 下載與安裝

### 下載執行檔

前往 [Releases](https://github.com/Yulin-9527/NIST_Analyzer/releases) 頁面下載最新版本的 `NIST_Analyzer.exe`

### 安裝步驟

本工具為免安裝版本：

1. 下載 `NIST_Analyzer.exe`
2. 將檔案放置於任意資料夾
3. 雙擊執行即可使用

> **注意**: 首次執行時，Windows SmartScreen 可能會出現警告訊息，請點選「更多資訊」→「仍要執行」

## 使用說明

### 基本操作

#### 方法一：拖曳檔案
1. 啟動 `NIST_Analyzer.exe`
2. 將 `.bin` 檔案拖曳到視窗中
3. 自動開始分析並顯示報告

#### 方法二：功能表開啟
1. 啟動程式
2. 點選功能表 **檔案 → 載入檔案** (或按 `Ctrl+O`)
3. 選擇要分析的 `.bin` 檔案
4. 檢視分析結果

### 報告內容說明

分析報告包含以下主要部分：

#### 1. 簽名驗證結果
顯示 NV_RSV、IBB、WBIO 三個區域的簽名驗證狀態：
- ✓ 通過 (綠色) - 簽名有效
- ✗ 失敗 (紅色) - 簽名無效或檔案損毀
- 若啟用 PDR 驗證，同時顯示 Backup NVRSV、Backup IBB、Backup WBIO 的驗證結果

#### 2. 結構摘要
快速瀏覽所有區塊的位址、大小和說明，點擊區塊名稱可跳轉至詳細內容

#### 3. 詳細資料內容
包含各區塊的完整十六進位數據：
- **Header 區塊**: BIOS 檔頭資訊 (1024 bytes)
- **RSA 公鑰**: 2048-bit RSA 公鑰 n 值 (256 bytes)
- **NV_RSV 資料區**: 保留配置區域及其簽名
- **IBB 資料區**: Initial Boot Block 及其簽名
- **WBIO 資料區**: Write Back I/O 及其簽名
- **PDR 備份區塊** (選用): 包含 Backup Descriptor、Backup NVRSV、Backup IBB、Backup WBIO 及其簽名

#### 4. 區塊分布圖
以表格形式呈現各區塊的位址範圍、大小和佔比

### 快捷鍵

| 快捷鍵 | 功能 |
|--------|------|
| `Ctrl+O` | 開啟檔案 |
| `Ctrl+Q` | 結束程式 |

### 進階設定

程式提供以下可自訂設定項目 (功能表：**設定 → 公鑰位址與PDR設定**)：

- **Public Key Address**: RSA 公鑰在 bin 檔案中的位址 (預設: 0x00034000)
- **啟用 PDR 驗證**: 是否解析並驗證 PDR 備份區塊 (預設: 停用)
- **PDR Base Address**: PDR 備份區域的基底位址 (預設: 0x00046000)

所有設定會自動儲存至 `config.json`，下次啟動時自動載入。

## 技術規格

### 支援的檔案格式
- NIST BIOS Bin 檔案 (`.bin`)

### 解析結構
- **Header**: 0x00000000 (1024 bytes)
- **RSA 公鑰**: 0x00034000 (256 bytes, 2048-bit)
- **NV_RSV**: 可變位址 (61440 bytes + 256 bytes 簽名)
- **IBB**: 可變位址與大小 (+ 256 bytes 簽名)
- **WBIO**: 可變位址與大小 (+ 256 bytes 簽名)

### 簽名驗證機制
採用 RSA-2048 PKCS#1 v1.5 驗證機制，使用 SHA-256 雜湊演算法

## 常見問題

### Q1: 為什麼我的 bin 檔案無法開啟？
**A**: 請確認：
- 檔案副檔名為 `.bin`
- 檔案符合 NIST BIOS 格式規範
- 檔案未損毀或加密

### Q2: 所有區域都顯示「✗ 失敗」是什麼原因？
**A**: 可能原因包括：
- 檔案格式不正確
- RSA 公鑰位址錯誤
- 檔案已被修改或損毀
- 不符合 NIST BIOS 規範

### Q3: 如何匯出分析報告？
**A**: 程式會自動在執行檔所在目錄生成 HTML 格式報告 (`report_檔名.html`)，並在預設瀏覽器中開啟

### Q4: PDR 驗證是什麼？我需要啟用嗎？
**A**: PDR (Platform Data Region) 是 BIOS 中的備份區域，包含 NVRSV、IBB、WBIO 的備份資料。若您的 bin 檔案包含 PDR 區域且需要驗證備份資料的完整性，可在設定中啟用此功能

### Q5: 如何修改公鑰位址？
**A**: 透過功能表 **設定 → 公鑰位址與PDR設定**，輸入新的位址（十六進位格式，如 0x00034000），儲存後重新載入檔案即可

## 技術支援

如果您遇到問題或有功能建議，請：

1. 查閱本文件的「常見問題」章節
2. 在 [Issues](https://github.com/Yulin-9527/NIST_Analyzer/issues) 頁面提交問題
3. 提供詳細的錯誤訊息和復現步驟

## 版本歷程

### v1.0.5 (2025-11-28)

- **新增 PDR 備份區塊驗證功能**
  - 支援解析 PDR (Platform Data Region) 備份區域
  - 驗證 Backup NVRSV、Backup IBB、Backup WBIO 簽名
  - 可選擇性啟用/停用 PDR 驗證
  - PDR 架構圖獨立顯示於報告中
- **新增設定檔系統**
  - 實作 `config.json` 設定檔持久化保存
  - 可自訂公鑰位址 (Public Key Address)
  - 可設定 PDR 基底位址 (PDR Base Address)
  - 設定於程式重啟後自動載入
- **介面優化**
  - 主畫面顯示當前公鑰位址與 PDR 驗證狀態
  - 新增「設定」選單，提供公鑰與 PDR 參數調整介面
  - PDR 驗證結果整合至摘要視窗
  - 報告新增 PDR 區塊頁籤
- **報告格式改進**
  - 將報告格式從 Markdown 改為 HTML
  - 報告自動在預設瀏覽器中開啟
  - 支援互動式頁籤切換 (主區塊/PDR 區塊)
  - 架構圖依位址自動排序顯示

### v1.0.1 (2025-11-10)

- 首次發布
- 支援 NIST BIOS Bin 檔案解析
- RSA-2048 簽名驗證功能
- 圖形化使用者介面
- 深色主題
- 拖放檔案支援
- 詳細分析報告生成
- 書籤導航功能

## 授權條款

本專案採用 MIT 授權條款 - 詳見 [LICENSE](LICENSE) 檔案

## 作者

**Yulin Hsieh**

- GitHub: [@Yulin-9527](https://github.com/Yulin-9527)

## 致謝

- 使用 [PyQt6](https://www.riverbankcomputing.com/software/pyqt/) 開發圖形介面
- 使用 [PyCryptodome](https://www.pycryptodome.org/) 實現 RSA 驗證
- 使用 [Mistune](https://github.com/lepture/mistune) 進行 Markdown 渲染

## 安全聲明

本工具僅用於合法的 BIOS 檔案分析與驗證用途。使用者應遵守所有適用的法律法規，不得將本工具用於任何非法目的。

---

**⭐ 如果這個專案對您有幫助，請給我們一個星星！**

#### [喜歡的話歡迎給個打賞吧!](https://ganknow.com/yulin9527/tip)
#### 歡迎用 BTC 支援：  
#### [bc1px0u84lk50jugxzwrcu26h2gsd00ef78pqplrr7sv96qgt8fgnrqqur0x7e](https://blockstream.info/address/bc1px0u84lk50jugxzwrcu26h2gsd00ef78pqplrr7sv96qgt8fgnrqqur0x7e)
#### 掃描 QR Code 進行 BTC 捐贈：
![BTC Donate QR Code](https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=bitcoin:bc1px0u84lk50jugxzwrcu26h2gsd00ef78pqplrr7sv96qgt8fgnrqqur0x7e)
