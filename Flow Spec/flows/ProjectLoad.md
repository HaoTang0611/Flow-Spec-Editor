# Flow Spec — 專案切換流程

**定位：語言無關 × 專案相關**

本文件定義系統在已就緒狀態下，操作員切換至不同專案配置的業務流程。

依賴：`IProjectProvider`、`ProjectConfiguration`、`ProjectMetadata`、`HardwareConfiguration`、`ICamera`、`ILightingController`、`ICommunicationChannel`、`IInspector`、`IAlarmReporter`、`IAlarmController`、`IAlarmProvider`、`ILogger`

治理規則：本文件所有警報觸發與日誌寫入行為 MUST 遵守 `cross-cutting-policy.md`。

---

## 一、流程定位

專案切換讓系統在不重啟的情況下載入不同的 `ProjectConfiguration`，並重新初始化對應的硬體資源。切換分為三個階段：釋放舊配置硬體資源、載入新專案配置、依新配置重新初始化硬體。

硬體初始化的個別步驟語意已由 `flows/Startup.md` 定義；本文件僅定義切換場景的前置條件、序列差異與失敗回應。

---

## 二、業務狀態定義

| 狀態 | 說明 |
|------|------|
| `Ready` | 專案已載入且硬體已就緒；業務操作可依各自前置條件執行（繼承自啟動流程） |
| `SwitchingProject` | 專案切換進行中；不接受任何業務操作 |
| `SwitchFailed` | 切換序列因必要步驟失敗而中止；不接受任何業務操作；需人工干預 |

---

## 三、狀態轉移

| 前置狀態 | 觸發條件 | 後繼狀態 | 副作用 |
|---------|---------|---------|-------|
| `Ready` | 操作員觸發專案切換且前置條件均滿足 | `SwitchingProject` | 記錄切換開始日誌（`Info`，含目標專案名稱） |
| `SwitchingProject` | 所有步驟成功完成 | `Ready` | 記錄切換完成日誌（`Info`，含新專案名稱） |
| `SwitchingProject` | 任一必要步驟失敗 | `SwitchFailed` | 記錄失敗原因日誌（`Error`）；觸發對應警報 |
| `SwitchFailed` | 系統重新啟動（見 `flows/Startup.md`） | `Ready`（依啟動結果） | 由啟動流程重新建立有效狀態 |

---

## 四、切換前置條件

觸發專案切換前，MUST 確認以下條件均已滿足；任一條件不滿足時 MUST NOT 進入 `SwitchingProject`：

1. 系統目前處於 `Ready` 狀態（非 `SwitchingProject` 或 `SwitchFailed`）
2. 系統業務狀態 MUST NOT 處於 `ErrorSuspended` 或 `CriticalFault`（見 `flows/AlarmHandling.md`）
3. 目前無進行中的批次推論作業（`IBatchInspector` 非執行中狀態）

關於工單狀態：若目前有進行中的工單（`WorkOrderActive`），MUST 先完成工單再執行切換（受 OQ-10 影響；若業務需求允許在工單進行中強制切換，需更新此前置條件並補充工單中止語意）。

---

## 五、專案切換序列

### 步驟 1：記錄切換開始

MUST 透過 `ILogger.Info` 記錄開始切換至目標專案（含目標專案名稱）。

---

### 步驟 2：釋放硬體資源

依下列順序中斷舊配置的硬體連線：

1. **斷開外部通訊**：若 `ICommunicationChannel` 處於連線狀態，呼叫 `Disconnect`
2. **斷開光源控制器**：對每個 `ILightingController` 實例呼叫 `Disconnect`
3. **斷開相機**：對每個 `ICamera` 實例呼叫 `Disconnect`

各步驟的 `Disconnect` 為清理型操作（不回傳結果型別）；任一步驟發生意外時，MUST 記錄 `Warning` 日誌並繼續執行後續步驟，釋放序列 MUST NOT 因單一步驟異常而中止。

> **推論模型**：`IInspector` 無卸載操作；舊模型狀態將在步驟 5 的 `LoadModel` 呼叫中被新配置取代，此處無需額外處理。

---

### 步驟 3：清除舊硬體關聯的 Active 警報

釋放資源後，MUST 清除所有與舊專案硬體裝置相關的 `Active` 警報（對每個相關警報識別碼呼叫 `IAlarmController.Clear`），確保新專案初始化後的警報環境乾淨。

---

### 步驟 4：載入新專案配置

呼叫 `IProjectProvider.Load`（以目標專案名稱為輸入）載入新的 `ProjectConfiguration`。

**成功**：取得新 `ProjectConfiguration`，繼續步驟 5。

**失敗**：依 `ProjectError` 類型記錄 `Error` 日誌（含錯誤代碼）；系統轉移至 `SwitchFailed`；切換序列終止。

---

### 步驟 5：依新配置重新初始化硬體

依新 `ProjectConfiguration.Hardware` 的配置，按 `flows/Startup.md` 步驟 3–6 的相同序列執行初始化：

1. **初始化相機**（步驟 3）：`Initialize` → `Connect` → `ApplyOptions`
2. **初始化光源控制器**（步驟 4）：`Connect` → `ApplyOptions`（各頻道）
3. **建立外部通訊連線**（步驟 5）：`Connect`（非必要，失敗不終止切換序列）
4. **載入推論模型**（步驟 6）：`LoadModel`（各已啟用的 Pipeline）

任一必要步驟失敗：MUST 記錄 `Error` 日誌（含裝置識別或 Pipeline 識別與錯誤代碼）；MUST 透過 `IAlarmReporter.Raise` 觸發對應警報；系統轉移至 `SwitchFailed`；序列終止。

各步驟的詳細失敗回應語意（含具體 `AlarmLevel`）依 `flows/Startup.md` 各步驟的規定執行。

---

### 步驟 6：重新訂閱連線中斷事件

重新初始化後，MUST 對新配置的硬體實例重新訂閱 `OnConnectionLost` 事件（依 `flows/Startup.md` 步驟 7 的規定執行）。

---

### 步驟 7：標記就緒

MUST 透過 `ILogger.Info` 記錄切換完成（含新專案名稱）；系統轉移至 `Ready`。

---

## 六、SwitchFailed 的業務回應

`SwitchFailed` 期間：

- 系統 MUST NOT 接受任何業務操作
- 不保證目前哪個（如果有）專案配置仍有效；硬體資源可能處於部分斷線或部分初始化狀態
- 操作員可重新嘗試切換（從步驟 1 重新執行），或重新啟動系統（`flows/Startup.md`）
- 重新嘗試或重啟的觸發條件由 Project Implementation 決定

---

## 七、邊界規則

| 責任 | 所屬層 |
|------|-------|
| 切換前是否需要先結束工單的最終業務決策 | 待 OQ-10 解決後更新本文件前置條件 |
| 正在進行中即時推論的中止語意（若切換觸發中止） | 依 OQ-06 解決結果適用 `flows/AlarmHandling.md` 規定 |
| 目標專案的選取方式（清單選擇、設定檔指定） | Project Implementation / UI 層 |
| 硬體實例（`ICamera`、`ILightingController` 等）在切換時是否重用舊實例或重建 | Project Implementation |
| `InspectionPipeline` 與 `IInspector` 實例、`ProjectModelConfiguration` 的對應機制 | Project Implementation（同 Startup 流程） |
| `SwitchFailed` 後的重試觸發介面 | Project Implementation / UI 層 |
| 硬體連線、斷線的具體驅動實作 | Core Implementation |
