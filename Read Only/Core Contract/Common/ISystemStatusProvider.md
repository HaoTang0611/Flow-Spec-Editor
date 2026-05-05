# ISystemStatusProvider

**定位：語言無關 × 專案無關**

本文件定義平台整體狀態查詢介面，提供目前系統健康狀態的讀取與變更通知能力。

依賴：引用 Common/SystemStatus

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `GetStatus` | 無 | `SystemStatus` | 回傳目前平台整體健康狀態 |

## 事件 / 回呼

| 事件名稱 | 資料 | 觸發時機 |
|---------|------|---------|
| `OnStatusChanged` | `SystemStatus` | 平台整體狀態發生變化時 |

## 約束

- `GetStatus` MUST 隨時可呼叫，不依賴任何前置操作
- `GetStatus` 回傳 `Normal` 時，MUST 保證平台無已知異常；回傳 `Warning` 時，MUST 保證存在至少一個非致命異常；回傳 `Error` 時，MUST 保證存在至少一個致命錯誤
- 聚合算法（如何由各子系統狀態決定整體狀態）由實作層定義，本層不描述
- `OnStatusChanged` MUST 僅在狀態值實際改變時觸發，不得重複觸發相同狀態
