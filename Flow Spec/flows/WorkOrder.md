# Flow Spec — 工單管理流程

**定位：語言無關 × 專案相關**

本文件定義工單識別碼（WorkOrderId）的業務生命週期，以及工單進行期間對推論輸入與結果歸屬的約束。

依賴：WorkOrder、InspectionInput、InspectionResult、ILogger

治理規則：本文件所有日誌寫入行為 MUST 遵守 `cross-cutting-policy.md`。

---

## 一、流程定位

工單管理流程定義本專案業務層中工單的起訖邊界與有效期間內的操作約束。工單是本專案業務操作的核心識別單元；推論結果透過 `WorkOrderId` 與其所屬工單關聯，供後續查詢與統計使用。系統在無工單狀態下亦可執行推論（如開發驗證情境），但結果不歸屬於任何工單。

---

## 二、業務狀態定義

| 狀態 | 說明 |
|------|------|
| `Idle` | 無進行中工單；系統可執行無工單推論（`InspectionInput.WorkOrderId` 為空值） |
| `WorkOrderActive` | 工單進行中；所有推論輸入 MUST 攜帶當前工單識別碼 |

---

## 三、狀態轉移

| 前置狀態 | 觸發條件 | 後繼狀態 | 副作用 |
|---------|---------|---------|-------|
| `Idle` | 工單識別碼確定且工單啟動 | `WorkOrderActive` | 記錄工單開始日誌（`Info`，含 `WorkOrderId`） |
| `WorkOrderActive` | 操作員主動結束工單 | `Idle` | 記錄工單結束日誌（`Info`，含 `WorkOrderId`） |
| `WorkOrderActive` | 系統關閉序列啟動（見 `flows/Shutdown.md`） | `Idle` | 記錄工單因系統關閉而中止日誌（`Info`，含 `WorkOrderId`）；依關閉流程繼續後續步驟 |

---

## 四、工單進行期間的業務約束

`WorkOrderActive` 期間 MUST 遵守：

- 所有推論操作的 `InspectionInput.WorkOrderId` MUST 填入當前工單識別碼，MUST NOT 為空值或空字串
- 推論成功後的 `InspectionResult.WorkOrderId` MUST 與當前工單識別碼一致
- `WorkOrderId` MUST NOT 為空字串（依 Core Contract WorkOrder 約束）

工單結束後（轉移至 `Idle`）：

- MUST NOT 繼續將新的推論結果歸屬於已結束的工單
- 後續推論請求若無新工單，`InspectionInput.WorkOrderId` MUST 為空值

---

## 五、工單終止流程（供 Shutdown 參照）

系統關閉序列觸發工單終止時，依序執行：

1. 停止接受歸屬於當前工單的新推論請求
2. 記錄工單因系統關閉而中止日誌（`Info`）
3. 清除當前工單識別碼，系統回到 `Idle`

---

## 六、邊界規則

| 責任 | 所屬層 |
|------|-------|
| `WorkOrderId` 的產生機制（手動輸入、通訊下達或系統自動產生） | Project Implementation |
| 工單啟動的觸發條件（使用者操作、通訊事件或其他外部訊號） | Project Implementation |
| 工單結束的觸發條件（使用者操作、通訊事件） | Project Implementation |
| `WorkOrderId` 唯一性保證 | Project Implementation |
| 推論輸入中 `WorkOrderId` 的填入（呼叫 `IInspector.Inspect` 時傳入） | Project Implementation（呼叫端） |
| 工單歷史查詢（`IResultRepository.ListWorkOrders`） | Core Contract（結果查詢層） |
