# ProjectConfiguration

**定位：語言無關 × 專案無關**

本文件定義專案聚合配置資料模型，集中描述一個專案的硬體設備、檢測流程、存檔策略與輸出入通道映射。

依賴：引用 Project/HardwareConfiguration、Project/InspectionPipelineConfiguration、Project/StorageConfiguration

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Hardware` | `HardwareConfiguration` | 硬體設備配置（多相機、通訊通道、光源控制器） |
| `InspectionPipelines` | `InspectionPipelineConfiguration` 列表 | 專案的檢測流程識別清單 |
| `Storage` | `StorageConfiguration` | 影像存檔策略配置 |
| `InputChannelMapping` | 字串→字串 映射 | 輸入通道識別碼與其用途名稱的對應表；空映射表示無輸入通道需求 |
| `OutputChannelMapping` | 字串→字串 映射 | 輸出通道識別碼與其用途名稱的對應表；空映射表示無輸出通道需求 |

## 約束

- `Hardware` MUST NOT 為空值
- `InspectionPipelines` MUST NOT 為空列表
- `Storage` MUST NOT 為空值
- `InputChannelMapping` 的鍵與值 MUST NOT 為空字串
- `OutputChannelMapping` 的鍵與值 MUST NOT 為空字串
