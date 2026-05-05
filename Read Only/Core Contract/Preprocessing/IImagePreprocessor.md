# IImagePreprocessor

**定位：語言無關 × 專案無關**

本文件定義影像前處理管線介面，依配置的步驟序列依序處理輸入影像。

依賴：引用 Common/ImageData、Preprocessing/PreprocessingStep、Preprocessing/PreprocessingOperationResult、Preprocessing/PreprocessingResult、Preprocessing/PreprocessingError

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Process` | `ImageData` | `PreprocessingResult` | 依配置的步驟序列對輸入影像執行前處理；影像為空時 MUST 回傳失敗，`Error` MUST 填入 `PreprocessingError.ImageEmpty` |
| `Configure` | `PreprocessingStep` 列表 | `PreprocessingOperationResult` | 設定前處理步驟序列；空列表表示不執行任何前處理，MUST 視為合法配置；若步驟類型或參數配置無效，MUST 回傳失敗，且 `Error` MUST 為 `PreprocessingError.ConfigurationInvalid` |
| `GetCurrentSteps` | 無 | `PreprocessingStep` 列表 | 回傳目前生效的步驟序列 |

## 約束

- `Process` MUST 依步驟列表順序依序執行，不得改變順序
- 若任一步驟執行失敗，`Process` MUST 回傳失敗，且 `Error` MUST 填入 `PreprocessingError.StepFailed`；`Detail` MUST 說明失敗步驟資訊
- 未呼叫 `Configure` 時，`GetCurrentSteps` MUST 回傳空列表
