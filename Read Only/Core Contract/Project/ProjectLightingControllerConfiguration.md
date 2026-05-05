# ProjectLightingControllerConfiguration

**定位：語言無關 × 專案無關**

本文件定義單一光源控制器於專案中的配置資料模型，用於表達控制器連線設定與其各頻道的預設參數。

依賴：引用 Communication/CommunicationOptions、Lighting/LightingOptions

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `ControllerId` | 字串 | 光源控制器在專案中的唯一識別碼 |
| `Communication` | `CommunicationOptions` | 此控制器的通訊連線設定 |
| `ChannelOptions` | 整數→`LightingOptions` 映射 | 各頻道的預設控制參數；空映射表示此控制器未預先配置任何頻道參數 |

## 約束

- `ControllerId` MUST NOT 為空字串
- `Communication` MUST NOT 為空值
- `ChannelOptions` 的每個鍵 MUST 對應目標控制器的一個頻道識別碼
- `ChannelOptions` 的每個值 MUST 符合 `LightingOptions` 定義
