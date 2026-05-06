# Flow Spec — 單次即時檢測流程

**定位：語言無關 × 專案相關**

本文件定義系統在就緒狀態下，針對即時取像資料執行一次完整推論並處理結果的業務序列。

依賴：IInspector、InspectionInput、InspectionInvokeResult、InspectionResult、Judgment、InspectionError、ICamera、CameraGrabResult、CameraTriggerMode、IImagePreprocessor、PreprocessingResult、PreprocessingError、IIOController、IOOperationResult、IOError、StorageConfiguration、ProjectConfiguration、ILogger

治理規則：本文件所有日誌寫入行為 MUST 遵守 `cross-cutting-policy.md`。

---

## 一、流程定位

單次即時檢測流程為本專案主要的線上作業模式，執行「取像→（前處理）→推論→結果處理」完整序列。此流程必須在系統就緒後（`flows/Startup.md`）方可觸發，且與工單狀態（`flows/WorkOrder.md`）共同決定推論結果的歸屬。

前置條件（每次觸發前 MUST 成立）：

| 條件 | 依賴 |
|------|------|
| 系統處於 `Ready` 狀態 | `flows/Startup.md` |
| 對應 Pipeline 的 `IInspector.IsModelLoaded` 為 `true` | `IInspector` |
| 相機 `CameraState` 為 `Connected` | `ICamera.GetState` |

---

## 二、業務狀態定義

以每個 InspectionPipeline 為單位定義狀態：

| 狀態 | 說明 |
|------|------|
| `Idle` | 等待觸發；可接受新的檢測請求 |
| `Inspecting` | 取像→推論序列進行中；MUST NOT 接受新的觸發，直至本次序列完成或失敗 |

---

## 三、狀態轉移

| 前置狀態 | 觸發條件 | 後繼狀態 | 副作用 |
|---------|---------|---------|-------|
| `Idle` | 觸發條件發生（見第四節）且前置條件成立 | `Inspecting` | — |
| `Inspecting` | 推論後處理全部完成（成功路徑） | `Idle` | — |
| `Inspecting` | 任一步驟失敗（取像 / 前處理 / 推論） | `Idle` | 依第七節記錄日誌 |

---

## 四、觸發條件

觸發方式依相機的 `CameraTriggerMode` 而定：

| CameraTriggerMode | 幀接收路徑 | 業務觸發時機 |
|-------------------|-----------|------------|
| `Software` | `GrabFrame` | 呼叫端明確請求取像；`GrabFrame` 回傳後啟動推論序列 |
| `Continuous` | `OnFrameArrived` | 相機產生新幀時收到事件；取得 `ImageData` 後啟動推論序列 |
| `Hardware` | `OnFrameArrived` | 外部硬體訊號觸發相機；收到 `OnFrameArrived` 事件後啟動推論序列 |

> **邊界**：觸發模式的選擇與觸發訊號的來源（使用者操作、系統排程、通訊事件）由 Project Implementation 決定，不屬於本層責任。

---

## 五、檢測序列

觸發條件發生且前置條件成立後，依序執行以下步驟。任一步驟失敗時，依第七節錯誤流程處理，不繼續後續步驟。

---

### 步驟 1：取得影像

| CameraTriggerMode | 動作 |
|-------------------|------|
| `Software` | 呼叫 `ICamera.GrabFrame`，取得 `CameraGrabResult`；`Success` 為 `false` 時視為取像失敗 |
| `Continuous` | 從 `OnFrameArrived` 事件接收 `ImageData`；此步驟已隱含於觸發路徑中 |
| `Hardware` | 從 `OnFrameArrived` 事件接收 `ImageData`；此步驟已隱含於觸發路徑中 |

---

### 步驟 2：前處理（若 Pipeline 配置了 IImagePreprocessor）

若該 InspectionPipeline 配置了 `IImagePreprocessor`：

1. 呼叫 `IImagePreprocessor.Process`，輸入步驟 1 取得的影像
2. 取得 `PreprocessingResult`
3. `Success` 為 `true`：以 `PreprocessingResult.Result` 作為後續推論的輸入影像
4. `Success` 為 `false`：視為前處理失敗，依第七節處理

若未配置 `IImagePreprocessor`，直接使用步驟 1 取得的影像。

---

### 步驟 3：組裝推論輸入

組裝 `InspectionInput`：

- `Image`：步驟 2 輸出的影像（或原始取像影像）；MUST NOT 為空值
- `WorkOrderId`：若系統目前處於 `WorkOrderActive` 狀態（`flows/WorkOrder.md`），MUST 填入當前工單識別碼；否則 MUST 為空值

---

### 步驟 4：執行推論

呼叫 `IInspector.Inspect`（輸入 `InspectionInput`），取得 `InspectionInvokeResult`：

- `Success` 為 `true`：取得 `InspectionResult`，繼續步驟 5
- `Success` 為 `false`：視為推論失敗，依第七節處理

---

### 步驟 5：推論後處理

`Judgment` 為唯一判定依據，MUST NOT 以 `Defects` 取代 `Judgment` 作為後續業務決策依據。

依 `InspectionResult.Judgment` 依序執行以下動作：

#### 5-a. IO 輸出

若 `ProjectConfiguration.OutputChannelMapping` 不為空，MUST 依映射關係對對應的輸出通道呼叫 `IIOController.WriteChannel`，以表達本次推論的 `Judgment`。

- `Judgment = Unknown` 的通道對應關係由 Project Implementation 定義；若無特別對應，SHOULD 依 NG 通道輸出（保守原則）
- `WriteChannel` 回傳失敗（`IOOperationResult.Success` 為 `false`）時：MUST 記錄 `Warning` 日誌（含通道識別與 `IOError` 代碼）；MUST NOT 因 IO 失敗中止存檔步驟

> **邊界**：`OutputChannelMapping` 中哪些通道對應 OK / NG / Unknown，及輸出訊號的復位時機，均由 Project Implementation 定義。

#### 5-b. 存檔決策

依 `ProjectConfiguration.Storage`（`StorageConfiguration`）決定是否保留原始影像：

| Judgment | 存檔條件 |
|----------|---------|
| `OK` | `StorageConfiguration.SaveRawImageOnOK` 為 `true` 時，MUST 觸發原始影像存檔 |
| `NG` | `StorageConfiguration.SaveRawImageOnNG` 為 `true` 時，MUST 觸發原始影像存檔 |
| `Unknown` | 依 `StorageConfiguration.SaveRawImageOnNG` 決定（保守原則） |

> **邊界**：原始影像的實際寫入（將 `ImageData` 持久化至儲存媒體）為 Project Implementation 責任；Flow Spec 只定義「在何種 `Judgment` 下應存檔」的判定條件。推論結果的持久化寫入時機亦由 Project Implementation 定義。

---

## 六、日誌要求

以下推論流程事件 MUST 透過 `ILogger` 寫入日誌（補充 `cross-cutting-policy.md` 未涵蓋的項目）：

| 事件 | LogLevel | 說明 |
|------|---------|------|
| 取像失敗（`CameraGrabResult.Success` 為 `false`） | `Warning` | 含相機識別與 `CameraError` 代碼 |
| 前處理失敗（`PreprocessingResult.Success` 為 `false`） | `Warning` | 含 Pipeline 識別與 `PreprocessingError` 代碼 |
| 推論失敗（`InspectionInvokeResult.Success` 為 `false`） | `Warning` | 含 Pipeline 識別與 `InspectionError` 代碼 |
| IO 輸出失敗 | `Warning` | 含通道識別與 `IOError` 代碼 |

---

## 七、錯誤流程

| 失敗步驟 | 業務回應 |
|---------|---------|
| 取像失敗 | 記錄 `Warning` 日誌；放棄本次檢測；Pipeline 返回 `Idle` |
| 前處理失敗 | 記錄 `Warning` 日誌；放棄本次檢測；Pipeline 返回 `Idle` |
| 推論失敗 | 記錄 `Warning` 日誌；放棄本次檢測；Pipeline 返回 `Idle` |

約束：

- 取像、前處理、推論的單次失敗均視為可恢復事件，MUST NOT 觸發 `AlarmLevel.Error` 或 `Critical` 警報
- Pipeline MUST 在失敗後返回 `Idle`，準備接受下一次觸發
- 硬體連線中斷引發的警報由 `cross-cutting-policy.md` 第二節定義，與單次流程失敗的回應分開處理

---

## 八、邊界規則

| 責任 | 所屬層 |
|------|-------|
| InspectionPipeline 與對應 `ICamera`、`IImagePreprocessor`、`IInspector` 實例的組合關係 | Project Implementation |
| 觸發方式的選擇與觸發訊號來源 | Project Implementation |
| 多個 Pipeline 的並行執行協調 | Project Implementation |
| 批次推論進行中是否允許本流程並行執行 | Project Implementation |
| `OutputChannelMapping` 中 Judgment 值與通道的對應規則 | Project Implementation |
| 輸出訊號的復位時機與策略 | Project Implementation |
| 原始影像的實際寫入實作 | Project Implementation |
| 推論結果的持久化寫入時機 | Project Implementation |
