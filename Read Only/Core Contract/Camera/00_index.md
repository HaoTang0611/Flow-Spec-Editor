# Camera — 索引

**定位：語言無關 × 專案無關**

本模組定義相機取像能力契約，包含連線管理、影像擷取與參數設定。

依賴：引用 Common/ImageData

---

## 文件清單

| 文件 | 說明 |
|------|------|
| [ICamera.md](ICamera.md) | 相機取像介面（連線、擷取、配置套用） |
| [DeviceInfo.md](DeviceInfo.md) | 相機裝置識別資料模型（Id、Name、Vendor） |
| [CameraState.md](CameraState.md) | 相機連線狀態枚舉 |
| [CameraTriggerMode.md](CameraTriggerMode.md) | 觸發模式枚舉（Pull / Push 主體與各模式行為約束） |
| [CameraOptions.md](CameraOptions.md) | 相機參數配置選項 |
| [CameraError.md](CameraError.md) | 相機操作錯誤語意 |
| [CameraOperationResult.md](CameraOperationResult.md) | 相機操作回傳結果（Initialize / Connect / ApplyOptions） |
| [CameraGrabResult.md](CameraGrabResult.md) | 影像擷取回傳結果（GrabFrame） |
