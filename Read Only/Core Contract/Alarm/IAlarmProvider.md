# IAlarmProvider

**定位：語言無關 × 專案無關**

本文件定義警報查詢與變更通知介面，提供目前警報清單與單筆警報查詢能力。

依賴：引用 Alarm/AlarmInfo、Alarm/AlarmState

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `ListByState` | `AlarmState` | `AlarmInfo` 列表 | 回傳指定狀態的警報清單 |
| `GetById` | 警報識別碼（字串） | `AlarmInfo` 或 空值 | 依警報識別碼取得單筆警報；不存在時 MUST 回傳空值 |

## 事件 / 回呼

| 事件名稱 | 資料 | 觸發時機 |
|---------|------|---------|
| `OnAlarmChanged` | `AlarmInfo` | 任一警報的狀態或內容發生變化時 |

## 約束

- `ListByState` 回傳的每筆警報，其 `State` MUST 與輸入值一致
- `OnAlarmChanged` MUST NOT 在警報的狀態與內容均未改變時重複觸發
- 警報因 `Clear` 導致狀態改變時，`OnAlarmChanged` MUST 觸發一次並回傳更新後的 `AlarmInfo`
