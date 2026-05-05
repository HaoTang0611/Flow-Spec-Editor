# ProjectError

**定位：語言無關 × 專案無關**

本文件定義專案配置操作錯誤語意，描述 `IProjectProvider` 各操作可能的失敗原因。

---

## 錯誤類型

| 錯誤名稱 | 觸發條件 |
|---------|---------|
| `ProjectNameInvalid` | `Load` 提供的專案名稱本身不合法（如空字串） |
| `ProjectNotFound` | `Load` 提供的專案名稱無法對應到任何可用專案資料夾 |
| `ContentReadFailed` | `Load` 無法讀取指定專案資料夾或其配置檔案 |
| `ConfigurationInvalid` | `Load` 載入的專案內容不符合專案配置規範 |
| `ContentIncomplete` | 專案配置缺少必要內容且無法建立預設值補足（如 `CameraOptions` 或必要模型設定欄位） |

## 約束

- 上述錯誤 MUST 在 `Load` 回傳失敗時可供呼叫端識別錯誤原因
