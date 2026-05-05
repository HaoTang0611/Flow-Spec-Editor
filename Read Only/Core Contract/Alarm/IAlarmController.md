# IAlarmController

**定位：語言無關 × 專案無關**

本文件定義警報狀態寫入介面，提供清除警報的能力。

依賴：引用 Alarm/AlarmState

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `Clear` | 警報識別碼（字串） | 布林 | 將指定警報狀態轉移至 `Cleared`；目標警報已為 `Cleared` 時視為成功並回傳 `true`；警報不存在時回傳 `false` |

## 約束

- 輸入識別碼 MUST NOT 為空字串
- 目標警報狀態為 `Active` 時，`Clear` MUST 將狀態更新為 `Cleared` 並回傳 `true`
- 目標警報狀態為 `Cleared` 時，`Clear` MUST 回傳 `true` 且 MUST NOT 修改任何警報狀態
- 目標警報不存在時，`Clear` MUST 回傳 `false` 且 MUST NOT 修改任何警報狀態
