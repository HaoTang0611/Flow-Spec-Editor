# ProjectModelConfiguration

**定位：語言無關 × 專案無關**

本文件定義單一模型於專案配置中的設定資料模型，供推論介面的模型載入方法作為輸入。

依賴：引用 Inspection/Judgment

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `IsEnabled` | 布林 | 此模型設定是否啟用 |
| `Name` | 字串 | 模型設定的顯示名稱 |
| `ModelReference` | 字串 | 模型識別資訊 |
| `Port` | 字串 | 模型對外使用的埠號或通道識別資訊 |
| `ConfidenceThreshold` | 浮點數 | 推論引擎產生 `Judgment` 時使用的最低信心分數；低於此值的結果 MUST 判定為 `Unknown` |

## 約束

- `Name` MUST NOT 為空字串
- `ModelReference` MUST NOT 為空字串
- `Port` MUST NOT 為空字串
- `ConfidenceThreshold` MUST 為 0.0 至 1.0 之間的值（含邊界）
- `IsEnabled` 為 `true` 時，表示此模型設定會被納入單次或批次檢查執行集合；是否可成功載入與執行仍取決於模型識別資訊、通道識別資訊與執行環境
- `IsEnabled` 為 `false` 的設定 MUST NOT 被視為可載入模型
