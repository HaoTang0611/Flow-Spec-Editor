# HardwareConfiguration

**定位：語言無關 × 專案無關**

本文件定義專案硬體設備配置聚合資料模型，集中描述多相機、通訊通道與多光源控制器的配置。

依賴：引用 Project/ProjectCameraConfiguration、Communication/CommunicationOptions、Project/ProjectLightingControllerConfiguration

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Cameras` | `ProjectCameraConfiguration` 列表 | 專案中的相機配置清單 |
| `CommunicationOptions` | `CommunicationOptions` 或 空值 | 專案的外部輸入與輸出通訊配置（如上位機）；若此專案無通訊需求可為空值 |
| `LightingControllers` | `ProjectLightingControllerConfiguration` 列表 | 專案中的光源控制器配置清單；空列表表示無光源控制需求 |

## 約束

- `Cameras` MUST NOT 為空列表
- `Cameras` 中的每筆資料 MUST 符合 `ProjectCameraConfiguration` 定義
- `Cameras` 中各 `CameraId` MUST 唯一
- `LightingControllers` 中的每筆資料 MUST 符合 `ProjectLightingControllerConfiguration` 定義
- `LightingControllers` 中各 `ControllerId` MUST 唯一
