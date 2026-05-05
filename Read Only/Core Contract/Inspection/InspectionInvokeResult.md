# InspectionInvokeResult

**定位：語言無關 × 專案無關**

本文件定義 `IInspector.Inspect` 操作的回傳型別，攜帶推論成功或失敗的完整語意。

依賴：引用 Inspection/InspectionResult、Inspection/InspectionError

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Success` | 布林 | `true` 表示推論成功完成；`false` 表示推論失敗 |
| `Error` | `InspectionError` 或 空值 | 失敗時填入對應錯誤碼；成功時 MUST 為空值 |
| `Result` | `InspectionResult` 或 空值 | 推論成功時填入結果；失敗時 MUST 為空值 |
| `Detail` | 字串 或 空值 | 補充診斷說明；僅供診斷用途，MUST NOT 作為程式流程判斷依據 |

## 約束

- `Success` 為 `true` 時，`Result` MUST NOT 為空值；`Error` MUST 為空值
- `Success` 為 `false` 時，`Error` MUST 填入 `InspectionError` 中定義的合法錯誤碼；`Result` MUST 為空值
