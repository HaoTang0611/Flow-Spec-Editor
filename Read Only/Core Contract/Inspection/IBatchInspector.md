# IBatchInspector

**定位：語言無關 × 專案無關**

本文件定義離線批次推論介面，針對呼叫端預先備妥的一組影像輸入（來源為已儲存的影像資料，非即時取像），依序執行推論、回報進度並支援取消操作。

依賴：引用 Inspection/InspectionInput、Inspection/InspectionResult、Inspection/BatchProgress、Inspection/BatchFailureInfo、Inspection/InspectionOperationResult、Inspection/InspectionError

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Start` | `InspectionInput` 列表 | `InspectionOperationResult` | 開始批次推論；模型未載入、輸入列表為空或已有批次執行中時 MUST 回傳失敗 |
| `Cancel` | 無 | 無 | 取消目前進行中的批次推論；未執行時呼叫 MUST 靜默完成 |
| `GetProgress` | 無 | `BatchProgress` 或 空值 | 查詢目前批次執行進度；批次未開始、已取消或因推論失敗中止時 MUST 回傳空值 |

## 事件 / 回呼

| 事件名稱 | 資料 | 觸發時機 |
|---------|------|---------|
| `OnInspectionCompleted` | `InspectionResult` | 每完成一筆推論並產生結果時 |
| `OnProgressUpdated` | `BatchProgress` | 每完成一筆推論後，進度數據更新時 |
| `OnBatchCompleted` | `BatchProgress` | 批次中所有推論均已成功完成時 |
| `OnBatchFailed` | `BatchFailureInfo` | 任一筆推論失敗且批次中止時 |
| `OnBatchCancelled` | `BatchProgress` 或 空值 | `Cancel` 執行完成後 |

## 約束

- `Start` 的輸入列表 MUST NOT 為空；空列表時 MUST 回傳失敗，且 `Error` MUST 為 `InspectionError.BatchInputEmpty`
- `Start` 呼叫時若模型尚未載入，MUST 回傳失敗，且 `Error` MUST 為 `InspectionError.ModelNotLoaded`
- `Start` 呼叫時若已有批次正在執行中，MUST 回傳失敗，不得中斷既有批次，且 `Error` MUST 為 `InspectionError.BatchAlreadyRunning`
- 每筆輸入成功完成推論後，`OnInspectionCompleted` MUST 觸發一次並回傳對應的 `InspectionResult`
- 任一筆推論失敗時，MUST 觸發 `OnBatchFailed` 並回傳對應的 `BatchFailureInfo`，MUST 立即中止批次，MUST NOT 繼續處理剩餘輸入
- `OnBatchCompleted` MUST 僅在批次正常全數完成時觸發；被 `Cancel` 中止或因推論失敗中止的批次 MUST NOT 觸發 `OnBatchCompleted`
- `Cancel` 執行完成後 MUST 觸發 `OnBatchCancelled`；批次未執行時呼叫 `Cancel`，`OnBatchCancelled` MUST NOT 觸發
- `Cancel` 呼叫後或批次因推論失敗中止後，`GetProgress` MUST 回傳空值
- `BatchProgress` 僅表示執行進度摘要；若實作層將批次結果保存供後續查詢，持久化內容與下游統計 MUST 以各筆 `InspectionResult` 為準，MUST NOT 僅由 `BatchProgress` 推導
- 批次執行所需的模型載入由呼叫端負責確保，本介面不定義模型載入方式
