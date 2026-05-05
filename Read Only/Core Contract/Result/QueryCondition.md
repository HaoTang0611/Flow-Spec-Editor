# QueryCondition

**定位：語言無關 × 專案無關**

本文件定義檢測結果查詢條件資料模型，供 `IResultRepository.Query` 與 `IResultRepository.Count` 使用。

依賴：引用 Inspection/Judgment、Common/Timestamp

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `WorkOrderId` | 字串 或 空值 | 依工單篩選；空值表示不限 |
| `Judgment` | `Judgment` 或 空值 | 依判定結果篩選；空值表示不限 |
| `From` | 時間戳記 或 空值 | 起始時間（含）；格式見 Common/Timestamp；空值表示不限 |
| `To` | 時間戳記 或 空值 | 結束時間（含）；格式見 Common/Timestamp；空值表示不限 |
| `PageIndex` | 整數 | 分頁索引，從 0 開始；僅對 `Query` 生效，`Count` 與 `GetStatistics` MUST 忽略此欄位 |
| `PageSize` | 整數 | 每頁筆數；0 表示不分頁（回傳全部符合筆數）；僅對 `Query` 生效，`Count` 與 `GetStatistics` MUST 忽略此欄位 |

## 約束

- `PageIndex` MUST 大於等於 0
- `PageSize` MUST 大於等於 0
- `From` 與 `To` 同時有值時，`From` MUST NOT 晚於 `To`
