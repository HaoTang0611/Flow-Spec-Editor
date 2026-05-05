# InspectionOperationResult

**定位：語言無關 × 專案無關**

本文件定義 Inspection 模組操作結果型別，供操作可能失敗且呼叫端需識別失敗原因的方法使用。

依賴：引用 Inspection/InspectionError

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Success` | 布林 | 操作是否成功 |
| `Error` | `InspectionError` 或 空值 | 失敗原因；`Success` 為 `true` 時 MUST 為空值 |
| `Detail` | 字串 或 空值 | 可選附加描述，僅供人工診斷使用 |

## 約束

- `Success` 為 `false` 時，`Error` MUST 填入 `InspectionError` 中定義的合法錯誤碼
- `Success` 為 `true` 時，`Error` MUST 為空值
- `Detail` 為選填；呼叫端 MUST NOT 依賴 `Detail` 進行邏輯判斷，僅供人工診斷使用
