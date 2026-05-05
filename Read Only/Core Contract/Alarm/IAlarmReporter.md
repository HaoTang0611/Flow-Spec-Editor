# IAlarmReporter

**定位：語言無關 × 專案無關**

本文件定義警報產生介面，提供平台核心能力回報新警報的入口。

依賴：引用 Alarm/AlarmInfo、Alarm/AlarmState

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Raise` | `AlarmInfo` | 布林 | 建立一筆新的 Active 警報；輸入不合法或警報識別碼已存在時回傳 `false` |

## 約束

- 輸入的 `AlarmInfo` MUST 符合 `AlarmInfo` 文件定義的所有約束
- 輸入的 `AlarmInfo.State` MUST 為 `Active`
- 輸入的 `AlarmInfo.AlarmId` 若已存在，`Raise` MUST 回傳 `false` 且 MUST NOT 修改既有警報
- `Raise` 成功時，該警報 MUST 可透過 `IAlarmProvider.GetById` 查詢取得
- `Raise` 成功時，`IAlarmProvider.OnAlarmChanged` MUST 觸發一次並回傳新增的 `AlarmInfo`
