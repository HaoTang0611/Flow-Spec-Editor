# AlarmInfo

**定位：語言無關 × 專案無關**

本文件定義警報資訊資料模型。

依賴：引用 Alarm/AlarmLevel、Alarm/AlarmState、Common/Timestamp

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `AlarmId` | 字串 | 警報唯一識別碼 |
| `Level` | `AlarmLevel` | 警報嚴重程度 |
| `State` | `AlarmState` | 警報目前狀態 |
| `Code` | 字串 | 警報代碼，用於識別警報類型 |
| `Message` | 字串 | 警報說明訊息 |
| `OccurredAt` | 時間戳記 | 警報發生時刻，格式見 Common/Timestamp |

## 約束

- `AlarmId` MUST NOT 為空字串
- `Code` MUST NOT 為空字串
- `Message` MUST NOT 為空字串
