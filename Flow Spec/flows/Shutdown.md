# Flow Spec — 系統關閉流程

**定位：語言無關 × 專案相關**

本文件定義系統從任意已啟動狀態（含 `Initializing`、`Ready`、`StartupFailed`）至完全關閉的資源釋放序列。

依賴：ICamera、ILightingController、ICommunicationChannel、ILogger、ProjectConfiguration

治理規則：本文件所有日誌寫入行為 MUST 遵守 `cross-cutting-policy.md`。

---

## 一、流程定位

系統關閉流程為應用程式退出前的必要序列，負責以正確順序釋放已取得的硬體資源，確保裝置在應用程式退出後回到穩定狀態。關閉序列 MUST 在所有業務操作停止後執行。

---

## 二、觸發條件

以下任一情況發生時，MUST 啟動關閉序列：

- 操作員主動觸發應用程式退出
- 作業系統發出應用程式終止信號

---

## 三、關閉序列

以下步驟依序執行。**關閉步驟中任一操作發生錯誤，MUST 記錄日誌但 MUST NOT 中止後續步驟**，確保所有已取得的資源均被嘗試釋放。

---

### 步驟 1：記錄關閉事件

MUST 透過 `ILogger.Info` 記錄系統關閉開始。

---

### 步驟 2：結束進行中的工單

若有工單正在進行中，MUST 先執行工單結束流程（詳見 `flows/WorkOrder.md`），再繼續後續步驟。

---

### 步驟 3：關閉外部通訊連線

若外部通訊通道已連線（`ICommunicationChannel.GetState` 為 `Connected`），呼叫 `Disconnect`。

---

### 步驟 4：關閉光源控制器

依 `ProjectConfiguration.Hardware.LightingControllers` 中的每台已配置光源控制器，依序執行：

1. 對每個已配置的頻道識別碼各呼叫一次 `TurnOff`，確保所有光源頻道已關閉
2. 呼叫 `Disconnect`

> `LightingControllers` 為空列表時，本步驟略過。

---

### 步驟 5：釋放相機資源

依 `ProjectConfiguration.Hardware.Cameras` 中的每台已配置相機，依序執行：

1. 呼叫 `Disconnect`
2. 呼叫 `Dispose`

---

### 步驟 6：關閉完成

MUST 透過 `ILogger.Info` 記錄系統關閉完成。

---

## 四、邊界規則

| 責任 | 所屬層 |
|------|-------|
| 關閉序列的觸發判定與使用者確認互動 | Project Implementation / UI 層 |
| `Disconnect` / `Dispose` 的具體驅動實作 | Core Implementation |
| 關閉時未完成工單的詳細處理邏輯 | `flows/WorkOrder.md`（Phase 2） |
| 光源控制器的哪些頻道識別碼需要呼叫 `TurnOff` | 依 `ProjectLightingControllerConfiguration` 中的頻道定義；Project Implementation 負責維護清單 |
| 部分初始化狀態下（`StartupFailed`）的釋放策略細節 | Project Implementation（需追蹤哪些資源已成功取得） |
