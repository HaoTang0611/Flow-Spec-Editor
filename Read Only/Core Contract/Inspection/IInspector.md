# IInspector

**定位：語言無關 × 專案無關**

本文件定義 AI 推論介面，輸入影像資料，輸出單次檢測結果。

依賴：引用 Inspection/InspectionInput、Inspection/InspectionResult、Inspection/InspectionInvokeResult、Inspection/InspectionOperationResult、Inspection/InspectionError、Inspection/Judgment、Project/ProjectModelConfiguration

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Inspect` | `InspectionInput` | `InspectionInvokeResult` | 對輸入執行推論；`Image` 為空時 MUST 回傳失敗，`Error` MUST 填入 `InspectionError.ImageEmpty` |
| `LoadModel` | `ProjectModelConfiguration` | `InspectionOperationResult` | 載入指定專案模型設定；設定停用、模型識別資訊無效或無法解析至可用模型時 MUST 回傳失敗 |
| `IsModelLoaded` | 無 | 布林 | 回傳目前是否已成功載入模型 |

## 約束

- 未呼叫 `LoadModel` 或載入失敗時，呼叫 `Inspect` MUST 回傳失敗，且 `Error` MUST 填入 `InspectionError.ModelNotLoaded`
- `InspectionInput.Image` 為空值時，`Inspect` MUST 回傳失敗，且 `Error` MUST 填入 `InspectionError.ImageEmpty`
- 推論過程發生無法恢復的內部錯誤時，`Inspect` MUST 回傳失敗，且 `Error` MUST 填入 `InspectionError.InferenceFailed`
- `LoadModel` 成功後，後續新的推論請求 MUST 依輸入 `ProjectModelConfiguration` 中的 `ConfidenceThreshold` 解讀模型輸出，並據此產生 `Judgment`
- `ProjectModelConfiguration.IsEnabled` 為 `false` 時，`LoadModel` MUST 回傳失敗，且 `Error` MUST 為 `InspectionError.ModelLoadFailed`
- `ProjectModelConfiguration.ModelReference` 無效或無法解析至可用模型時，`LoadModel` MUST 回傳失敗，且 `Error` MUST 為 `InspectionError.ModelLoadFailed`
- `LoadModel` 僅影響後續新的推論請求，對已接受處理的推論請求 MUST NOT 改變其結果語意
- `Inspect` 為單次推論的直接回傳介面；完成語意以方法回傳的 `InspectionInvokeResult` 為準，本介面不另外定義完成事件
