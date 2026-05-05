# PreprocessingError

**定位：語言無關 × 專案無關**

本文件定義影像前處理操作錯誤語意，描述 `IImagePreprocessor` 各操作可能的失敗原因。

---

## 錯誤類型

| 錯誤名稱 | 觸發條件 |
|---------|---------|
| `ImageEmpty` | 呼叫 `Process` 時輸入影像為空值 |
| `ConfigurationInvalid` | 呼叫 `Configure` 時步驟類型非法、必要參數缺失或參數內容無法被接受 |
| `StepFailed` | `Process` 執行過程中任一步驟發生無法恢復的處理失敗 |

## 約束

- `Process` 回傳失敗時，`Error` MUST 可對應至上述錯誤類型之一
- `Configure` 回傳失敗時，`Error` MUST 可對應至上述錯誤類型之一
