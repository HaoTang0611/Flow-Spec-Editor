# IOError

**定位：語言無關 × 專案無關**

本文件定義 IO 通道操作錯誤語意，描述 `IIOController` 各操作可能的失敗原因。

---

## 錯誤類型

| 錯誤名稱 | 觸發條件 |
|---------|---------|
| `ChannelNotFound` | 指定通道識別碼不存在於已知通道列表中 |
| `DirectionNotAllowed` | 對 `Input` 方向通道執行寫入操作 |
| `ControllerUnavailable` | IO 控制器尚未就緒或連線已中斷 |
| `WriteFailed` | 硬體層回報電位設定未完成 |

## 約束

- 各錯誤碼 MUST 作為 `IOOperationResult.Error` 的填入值回傳給呼叫端
- 下表說明各錯誤碼與觸發方法的對應關係（僅限回傳 `IOOperationResult` 的方法）：

| 錯誤名稱 | 可能觸發的方法 |
|---------|--------------|
| `ChannelNotFound` | `WriteChannel` |
| `DirectionNotAllowed` | `WriteChannel` |
| `ControllerUnavailable` | `WriteChannel` |
| `WriteFailed` | `WriteChannel` |

- `ReadChannel` 與 `ListChannels` 不回傳 `IOOperationResult`；其控制器未就緒或通道不存在的語意以空值／空列表表達，詳見 `IIOController`
- 具體附帶資訊（如通道識別碼）MUST 填入 `IOOperationResult.Detail` 欄位
