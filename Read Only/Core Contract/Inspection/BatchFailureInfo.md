# BatchFailureInfo

**定位：語言無關 × 專案無關**

本文件定義批次推論失敗事件資料模型，封裝導致批次中止的失敗輸入、錯誤語意與中止時進度。

依賴：引用 Inspection/InspectionInput、Inspection/InspectionError、Inspection/BatchProgress

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `FailedInput` | `InspectionInput` | 導致批次中止的推論輸入 |
| `Error` | `InspectionError` | 導致批次中止的錯誤語意 |
| `Progress` | `BatchProgress` | 批次中止時的進度摘要 |

## 約束

- `FailedInput` MUST NOT 為空值
- `Error` MUST 為 `InspectionError` 的合法值
- `Progress` MUST NOT 為空值
