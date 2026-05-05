# Core Contract — 索引

**定位：語言無關 × 專案無關**

本目錄定義平台核心的介面契約、資料模型與配置契約。所有描述均與實作語言及具體專案無關，任何語言的開發者皆可依此文件實作或驗證符合性。

依賴規則：

- 語言相關層（Core Implementation、WPF Implementation、Project Implementation）MUST 依賴本層定義，反向不行。
- 本層 MUST NOT 引用任何語言相關或專案相關的概念。

---

## 模組清單

| 模組 | 說明 |
|------|------|
| [Common/](Common/00_index.md) | 跨模組共用的基礎語意與型別 |
| [Account/](Account/00_index.md) | 帳號登入與使用者管理能力契約 |
| [Camera/](Camera/00_index.md) | 相機取像能力契約 |
| [Communication/](Communication/00_index.md) | 通訊通道能力契約 |
| [Inspection/](Inspection/00_index.md) | AI 推論能力契約 |
| [Preprocessing/](Preprocessing/00_index.md) | 影像前處理管線能力契約 |
| [Project/](Project/00_index.md) | 專案配置讀取能力契約 |
| [Result/](Result/00_index.md) | 檢測結果查詢能力契約 |
| [Alarm/](Alarm/00_index.md) | 警報資訊與取用能力契約 |
| [IO/](IO/00_index.md) | 數位 IO 通道讀寫能力契約 |
| [Lighting/](Lighting/00_index.md) | 光源控制器能力契約 |
| [Logging/](Logging/00_index.md) | 系統日誌寫入與查詢能力契約 |
