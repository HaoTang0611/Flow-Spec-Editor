# Flow Spec — 索引

**定位：語言無關 × 專案相關**

本目錄定義本專案業務流程的語言無關規格。所有內容以 Core Contract 為依賴基礎，描述本專案特有的業務場景、狀態轉移與觸發條件。

依賴規則：

- 本層 MUST 依賴 Core Contract 定義的介面、資料模型與操作語意
- 本層 MUST NOT 包含任何語言語法、框架語意或實作細節
- Project Implementation MUST 依賴本層定義作為業務流程的唯一依據；反向不行

---

## 文件清單

### 跨流程治理

| 文件 | 說明 |
|------|------|
| [cross-cutting-policy.md](cross-cutting-policy.md) | 跨流程通用治理規則：AlarmLevel 業務回應策略、硬體斷線警報觸發規則、日誌寫入規則、工單語意、SystemStatus 與業務流程關聯 |

### 系統生命週期

| 文件 | 說明 |
|------|------|
| [flows/Startup.md](flows/Startup.md) | 系統啟動流程：專案載入、相機初始化、光源控制器初始化、通訊連線、推論模型載入至系統就緒 |
| [flows/Shutdown.md](flows/Shutdown.md) | 系統關閉流程：資源有序釋放序列 |

### 核心作業流程

| 文件 | 說明 |
|------|------|
| [flows/WorkOrder.md](flows/WorkOrder.md) | 工單管理流程：工單生命週期（Idle ↔ WorkOrderActive）、推論輸入的工單歸屬約束 |
| [flows/Inspection.md](flows/Inspection.md) | 單次即時檢測流程：觸發取像→（前處理）→推論→IO 輸出→存檔決策完整序列 |
| [flows/BatchInspection.md](flows/BatchInspection.md) | 離線批次檢測流程：批次啟動、進度監視、結果處理、取消與失敗回應 |

### 輔助作業流程

| 文件 | 說明 |
|------|------|
| `flows/AlarmHandling.md` | 警報觸發與清除流程（待建立） |
| `flows/AccountSession.md` | 帳號登入登出流程（待建立） |
| `flows/ProjectLoad.md` | 專案切換流程（待建立） |
