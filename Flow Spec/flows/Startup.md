# Flow Spec — 系統啟動流程

**定位：語言無關 × 專案相關**

本文件定義系統從應用程式啟動至進入業務就緒狀態的完整序列。

依賴：IProjectProvider、ProjectConfiguration、HardwareConfiguration、ICamera、CameraOperationResult、ILightingController、LightingOperationResult、ICommunicationChannel、CommunicationOperationResult、IInspector、InspectionOperationResult、IAlarmReporter、ILogger

治理規則：本文件所有警報觸發與日誌寫入行為 MUST 遵守 `cross-cutting-policy.md`。

---

## 一、流程定位

系統啟動流程為所有業務操作的必要前置條件。啟動序列完成且系統進入就緒狀態後，其他業務流程（WorkOrder、Inspection 等）方可執行。

---

## 二、業務狀態定義

| 狀態 | 說明 |
|------|------|
| `Initializing` | 啟動序列進行中；不接受任何業務操作 |
| `Ready` | 所有必要資源已就緒；業務操作可依各自前置條件執行 |
| `StartupFailed` | 啟動序列因必要步驟失敗而中止；不接受任何業務操作；需人工干預後重新啟動 |

---

## 三、狀態轉移

| 前置狀態 | 觸發條件 | 後繼狀態 | 副作用 |
|---------|---------|---------|-------|
| 初始（應用程式進入主執行流程） | — | `Initializing` | 記錄啟動開始日誌（`Info`） |
| `Initializing` | 所有必要步驟成功完成 | `Ready` | 記錄就緒日誌（`Info`） |
| `Initializing` | 任一必要步驟失敗 | `StartupFailed` | 記錄失敗原因日誌（`Error`）；觸發對應警報 |

---

## 四、啟動序列

以下步驟依序執行。**任一必要步驟失敗時，啟動序列 MUST 立即終止**，系統轉移至 `StartupFailed`，除非該步驟另有說明（如：通訊連線為非必要步驟）。

---

### 步驟 1：記錄啟動事件

MUST 透過 `ILogger.Info` 記錄系統啟動開始。

---

### 步驟 2：載入專案

當專案名稱已確定後，呼叫 `IProjectProvider.Load`，載入指定專案的完整配置，取得 `ProjectConfiguration`。

**成功**：取得 `ProjectConfiguration`，繼續步驟 3。

**失敗**：依錯誤類型處理如下：

| ProjectError | 業務回應 |
|-------------|---------|
| `ProjectNotFound` | 記錄 `Error` 日誌；系統轉移至 `StartupFailed` |
| `ConfigurationInvalid` | 記錄 `Error` 日誌；系統轉移至 `StartupFailed` |
| `ContentIncomplete` | 記錄 `Error` 日誌；系統轉移至 `StartupFailed` |
| `ContentReadFailed` | 記錄 `Error` 日誌；系統轉移至 `StartupFailed` |

> **邊界**：啟動時使用哪個專案（上次使用的專案、使用者選擇、或預設值）由 Project Implementation 決定，不屬於本層責任。

---

### 步驟 3：初始化相機

依 `ProjectConfiguration.Hardware.Cameras` 中的每筆 `ProjectCameraConfiguration`，對對應的 `ICamera` 實例依序執行：

1. `Initialize`
2. `Connect`
3. `ApplyOptions`（以該 `ProjectCameraConfiguration` 中的相機參數為輸入）

任一子步驟回傳失敗時：MUST 記錄 `Error` 日誌（含裝置識別與錯誤代碼）；MUST 透過 `IAlarmReporter.Raise` 觸發 `AlarmLevel.Error` 警報；啟動序列 MUST 終止，系統轉移至 `StartupFailed`。

每台相機三個子步驟全部成功後，MUST 記錄 `Info` 日誌（含裝置識別）。

---

### 步驟 4：初始化光源控制器

依 `ProjectConfiguration.Hardware.LightingControllers` 中的每筆 `ProjectLightingControllerConfiguration`，對對應的 `ILightingController` 實例依序執行：

1. `Connect`（以該 `ProjectLightingControllerConfiguration` 中的通訊選項為輸入）
2. 對每個已配置的頻道識別碼各呼叫一次 `ApplyOptions`（以對應頻道的 `LightingOptions` 為輸入）

任一子步驟回傳失敗時：MUST 記錄 `Error` 日誌（含裝置識別與錯誤代碼）；MUST 透過 `IAlarmReporter.Raise` 觸發 `AlarmLevel.Error` 警報；啟動序列 MUST 終止，系統轉移至 `StartupFailed`。

每台光源控制器成功完成後，MUST 記錄 `Info` 日誌（含裝置識別）。

> `ProjectConfiguration.Hardware.LightingControllers` 為空列表時，本步驟略過。

---

### 步驟 5：建立外部通訊連線

若 `ProjectConfiguration.Hardware.CommunicationOptions` 不為空值，對 `ICommunicationChannel` 實例呼叫 `Connect`（以 `CommunicationOptions` 為輸入）。

**成功**：記錄 `Info` 日誌；繼續步驟 6。

**失敗**：記錄 `Warning` 日誌（含錯誤代碼）；MUST 透過 `IAlarmReporter.Raise` 觸發 `AlarmLevel.Warning` 警報；**通訊連線失敗 MUST NOT 終止啟動序列**，繼續執行步驟 6。

> 本步驟採非必要設計：通訊通道失敗不阻止系統就緒，但業務層 MUST 記錄此狀態。若通訊對業務流程的影響超出此政策（如：WorkOrder 強依賴通訊），由 `flows/WorkOrder.md` 定義追加約束。

> `ProjectConfiguration.Hardware.CommunicationOptions` 為空值時，本步驟略過。

---

### 步驟 6：載入推論模型

本專案的每個 `InspectionPipeline` MUST 在啟動時完成其對應 `IInspector` 實例的模型載入。對每個已配置且啟用的 Pipeline，呼叫對應 `IInspector` 實例的 `LoadModel`（以對應的 `ProjectModelConfiguration` 為輸入）。

任一 `LoadModel` 回傳失敗時：MUST 記錄 `Error` 日誌（含 Pipeline 識別與 `InspectionError` 代碼）；MUST 透過 `IAlarmReporter.Raise` 觸發 `AlarmLevel.Error` 警報；啟動序列 MUST 終止，系統轉移至 `StartupFailed`。

`ProjectModelConfiguration.IsEnabled` 為 `false` 的模型設定 MUST 略過，不呼叫 `LoadModel`。

每個 Pipeline 模型載入成功後，MUST 記錄 `Info` 日誌。

> **邊界**：各 `InspectionPipelineConfiguration` 與其對應 `IInspector` 實例、`ProjectModelConfiguration` 的關聯機制由 Project Implementation 定義，不屬於本層責任。

---

### 步驟 7：訂閱連線中斷事件

系統進入 `Ready` 狀態前，MUST 訂閱以下事件，以處理執行期間的硬體連線中斷：

| 事件來源 | 事件 | 業務回應 |
|---------|------|---------|
| 每個 `ICamera` 實例 | `OnConnectionLost` | 觸發 `AlarmLevel.Error` 警報；記錄 `Error` 日誌 |
| 每個 `ILightingController` 實例 | `OnConnectionLost` | 觸發 `AlarmLevel.Error` 警報；記錄 `Error` 日誌 |
| `ICommunicationChannel`（若已連線） | `OnConnectionLost` | 觸發 `AlarmLevel.Warning` 警報；記錄 `Error` 日誌 |

---

### 步驟 8：標記就緒

MUST 透過 `ILogger.Info` 記錄系統進入就緒狀態；系統轉移至 `Ready`。

---

## 五、邊界規則

| 責任 | 所屬層 |
|------|-------|
| 啟動時使用哪個專案的選擇機制 | Project Implementation / UI 層 |
| `ICamera`、`ILightingController`、`IInspector` 等實例的建立與注入 | Project Implementation |
| InspectionPipeline 與 IInspector 實例、ProjectModelConfiguration 的對應機制 | Project Implementation |
| 硬體 Initialize / Connect / ApplyOptions 的具體驅動實作 | Core Implementation |
| 啟動失敗時的使用者通知方式 | Project Implementation / UI 層 |
| StartupFailed 後的重試觸發條件 | Project Implementation / UI 層 |
