# InspectionError

**定位：語言無關 × 專案無關**

本文件定義 Inspection 模組操作錯誤語意，描述 `IInspector` 與 `IBatchInspector` 各操作可能的失敗原因。

---

## 錯誤類型

| 錯誤名稱 | 觸發條件 |
|---------|---------|
| `ModelNotLoaded` | 呼叫 `Inspect` 時模型尚未載入 |
| `ModelLoadFailed` | `LoadModel` 提供的模型設定停用、模型識別資訊無效，或無法對應到可用模型 |
| `ImageEmpty` | `Inspect` 傳入的 `InspectionInput.Image` 為空值 |
| `InferenceFailed` | 推論過程中發生無法恢復的內部錯誤 |
| `BatchAlreadyRunning` | 呼叫 `Start` 時已有另一個批次正在執行中 |
| `BatchInputEmpty` | 呼叫 `Start` 時輸入列表為空 |

## 約束

- `Inspect` 回傳失敗時，`Error` MUST 可對應至上述錯誤類型之一
- `LoadModel` 或 `Start` 回傳失敗時，失敗原因 MUST 可對應至上述錯誤類型之一
