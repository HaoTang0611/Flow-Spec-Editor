# IOChannelState

**定位：語言無關 × 專案無關**

本文件定義單一 IO 通道的即時狀態資料模型。

依賴：引用 IO/IODirection

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `ChannelId` | 字串 | 通道唯一識別碼 |
| `Direction` | `IODirection` | 通道方向（Input 或 Output） |
| `IsActive` | 布林 | 通道目前是否為高電位（布林為真表示高電位） |

## 約束

- `ChannelId` MUST NOT 為空字串
- `Direction` MUST 為合法的 `IODirection` 枚舉值
