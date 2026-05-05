# IIOController

**定位：語言無關 × 專案無關**

本文件定義 IO 通道讀寫控制介面，提供數位 IO 通道的讀取、寫入與即時狀態查詢能力。

依賴：引用 IO/IOChannelState、IO/IOOperationResult、IO/IOError

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `ReadChannel` | 通道識別碼（字串） | `IOChannelState` 或 空值 | 讀取指定通道的即時狀態；通道不存在時 MUST 回傳空值 |
| `WriteChannel` | 通道識別碼（字串）、啟用狀態（布林） | `IOOperationResult` | 設定指定輸出通道的電位狀態；失敗原因見 `IOError` |
| `ListChannels` | 無 | `IOChannelState` 列表 | 回傳所有已知通道的即時狀態 |

## 事件 / 回呼

| 事件名稱 | 資料 | 觸發時機 |
|---------|------|---------|
| `OnChannelStateChanged` | `IOChannelState` | 任一通道狀態發生變化時 |

## 約束

- `WriteChannel` 對 `Input` 方向通道 MUST 回傳 `IOOperationResult`，其中 `Error` 填入 `IOError.DirectionNotAllowed`，不得改變通道狀態
- `WriteChannel` 對不存在的通道 MUST 回傳 `IOOperationResult`，其中 `Error` 填入 `IOError.ChannelNotFound`
- 若控制器未就緒，`WriteChannel` MUST 回傳 `IOOperationResult`，其中 `Error` 填入 `IOError.ControllerUnavailable`
- 若控制器未就緒，`ReadChannel` MUST 回傳空值；`ListChannels` MUST 回傳空列表
