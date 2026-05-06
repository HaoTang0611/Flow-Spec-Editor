# Flow Spec — 離線批次檢測流程

**定位：語言無關 × 專案相關**

本文件定義針對預先備妥的已儲存影像資料集執行離線批次推論的業務序列。

依賴：IBatchInspector、InspectionInput、InspectionResult、BatchProgress、BatchFailureInfo、InspectionOperationResult、InspectionError、Judgment、ILogger

治理規則：本文件所有日誌寫入行為 MUST 遵守 `cross-cutting-policy.md`。

---

## 一、流程定位

離線批次推論使用呼叫端預先備妥的已儲存影像資料（非即時取像），因此不涉及相機取像、光源控制與 IO 輸出，與單次即時檢測（`flows/Inspection.md`）的執行路徑完全獨立。

兩者共用相同的推論模型層（`IInspector` 實例），但透過 `IBatchInspector` 介面執行批次操作。批次期間的並行控制詳見第五節。

前置條件（啟動批次前 MUST 成立）：

| 條件 | 依賴 |
|------|------|
| 系統處於 `Ready` 狀態 | `flows/Startup.md` |
| `IInspector.IsModelLoaded` 為 `true` | `IInspector` |
| 無其他批次正在執行中（`IBatchInspector.GetProgress` 為空值） | `IBatchInspector` |

---

## 二、業務狀態定義

| 狀態 | 說明 |
|------|------|
| `Idle` | 無批次進行中；可接受新的批次請求 |
| `BatchRunning` | 批次推論進行中；MUST NOT 接受新的批次啟動請求 |

---

## 三、狀態轉移

| 前置狀態 | 觸發條件 | 後繼狀態 | 副作用 |
|---------|---------|---------|-------|
| `Idle` | `IBatchInspector.Start` 回傳成功 | `BatchRunning` | 記錄批次開始日誌（`Info`，含輸入張數與 `WorkOrderId`（若存在）） |
| `BatchRunning` | `OnBatchCompleted` 觸發 | `Idle` | 記錄批次完成日誌（`Info`，含總張數與 `WorkOrderId`（若存在）） |
| `BatchRunning` | `OnBatchFailed` 觸發 | `Idle` | 記錄批次失敗日誌（`Error`，含 `InspectionError` 代碼與已完成張數） |
| `BatchRunning` | `OnBatchCancelled` 觸發 | `Idle` | 記錄批次取消日誌（`Info`，含已完成張數） |

---

## 四、批次啟動序列

### 步驟 1：確認前置條件

呼叫 `IBatchInspector.Start` 前，MUST 確認：

1. `IInspector.IsModelLoaded` 為 `true`
2. 輸入列表 MUST NOT 為空
3. `IBatchInspector.GetProgress` 為空值（無批次執行中）

任一條件不滿足時，MUST NOT 呼叫 `Start`；MUST 記錄 `Warning` 日誌（含不滿足的條件描述）。

### 步驟 2：組裝輸入列表

批次輸入（`InspectionInput` 列表）的 `WorkOrderId` MUST 遵守以下規則：

- 系統目前處於 `WorkOrderActive` 狀態（`flows/WorkOrder.md`）：所有輸入的 `WorkOrderId` MUST 填入當前工單識別碼
- 系統目前處於 `Idle`（無工單）：所有輸入的 `WorkOrderId` MUST 為空值

### 步驟 3：啟動批次

呼叫 `IBatchInspector.Start`（輸入組裝完成的 `InspectionInput` 列表），取得 `InspectionOperationResult`：

- `Success` 為 `true`：系統轉移至 `BatchRunning`，依第五節監視執行
- `Success` 為 `false`：記錄 `Warning` 日誌（含 `InspectionError` 代碼）；系統保持 `Idle`

---

## 五、批次執行期間的約束

`BatchRunning` 期間 MUST 遵守：

- MUST NOT 呼叫 `IBatchInspector.Start` 啟動新批次
- 即時單次推論（`flows/Inspection.md`）與批次並行的行為尚未決策，待業務需求確認後補充定義

---

## 六、批次進度與結果事件

批次執行期間，MUST 訂閱並處理以下事件：

| 事件 | 業務回應 |
|------|---------|
| `OnInspectionCompleted`（每筆推論完成） | 依第七節處理各筆結果 |
| `OnProgressUpdated` | 進度呈現由 Project Implementation 負責；Flow Spec 不定義進度 UI 行為 |
| `OnBatchCompleted` | 依第三節狀態轉移處理；記錄完成日誌 |
| `OnBatchFailed` | 依第三節與第八節錯誤流程處理；記錄失敗日誌 |
| `OnBatchCancelled` | 依第三節狀態轉移處理；記錄取消日誌 |

---

## 七、批次結果處理

每次 `OnInspectionCompleted` 觸發時，對取得的 `InspectionResult`：

- MUST 以 `Judgment` 為唯一判定依據（`Judgment` 為 `OK` / `NG` / `Unknown`）；MUST NOT 以 `Defects` 取代 `Judgment` 作為業務決策依據
- 推論結果的持久化寫入由 Project Implementation 定義（批次情境下的時機由呼叫端決定）

批次推論不涉及即時 IO 輸出（輸入為已儲存影像，無線上物件等待判定訊號）。

批次推論的存檔決策：批次輸入影像為已儲存資料，Flow Spec 不要求重複存檔；Project Implementation 可依業務需求定義是否另行保留批次推論的原始影像副本。

---

## 八、取消流程

使用者或系統可於 `BatchRunning` 狀態隨時取消批次：

1. 呼叫 `IBatchInspector.Cancel`
2. 等待 `OnBatchCancelled` 事件（Core Contract 保證 `Cancel` 完成後觸發）
3. 收到 `OnBatchCancelled` 後：系統轉移至 `Idle`；記錄取消日誌（`Info`，含已完成張數）

約束：

- 批次未執行時呼叫 `Cancel`，`OnBatchCancelled` MUST NOT 觸發；系統保持 `Idle`
- 取消後 MUST NOT 繼續接受已取消批次的結果事件

---

## 九、錯誤流程

### 批次啟動失敗

`IBatchInspector.Start` 回傳失敗時：

- 記錄 `Warning` 日誌（含 `InspectionError` 代碼）
- 系統保持 `Idle`，不轉移至 `BatchRunning`

### 批次中止（OnBatchFailed）

收到 `OnBatchFailed`（含 `BatchFailureInfo`）時：

- 記錄 `Error` 日誌（含 `BatchFailureInfo.Error`、`BatchFailureInfo.Progress.ProcessedCount`）
- 系統轉移至 `Idle`
- Core Contract 保證 `OnBatchFailed` 後不再觸發剩餘輸入的 `OnInspectionCompleted`；業務層 MUST NOT 繼續等待後續結果

---

## 十、邊界規則

| 責任 | 所屬層 |
|------|-------|
| 批次輸入列表的建立（選擇哪些已儲存影像、以何種順序） | Project Implementation |
| 批次執行中的進度 UI 呈現 | Project Implementation |
| 批次結果的持久化寫入時機 | Project Implementation |
| 批次推論期間即時單次推論的互斥保證 | 尚未決策；待業務需求確認後補充定義 |
| 批次推論是否觸發 IO 輸出 | 不屬於本層規範（批次為離線模式，無線上 IO 需求） |
| 批次輸入影像的重複存檔決策 | Project Implementation |
