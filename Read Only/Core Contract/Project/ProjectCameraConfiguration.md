# ProjectCameraConfiguration

**定位：語言無關 × 專案無關**

本文件定義單一相機於專案中的配置資料模型，描述相機識別與硬體參數。

依賴：引用 Camera/CameraOptions、Camera/DeviceInfo

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `CameraId` | 字串 | 目標相機識別碼 |
| `DisplayName` | 字串 | 使用者可識別名稱（例：上相機、側相機） |
| `Options` | `CameraOptions` | 此相機的參數配置 |

## 約束

- `CameraId` MUST NOT 為空字串
- `CameraId` MUST 與對應 `DeviceInfo.Id` 相符
- `DisplayName` MUST NOT 為空字串
- `Options` MUST NOT 為空值
