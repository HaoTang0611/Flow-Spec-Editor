# Flow Spec — 跨流程通用治理政策

**定位：語言無關 × 專案相關**

本文件定義適用於本專案所有業務流程的通用治理規則。所有流程文件 MUST 遵守本政策；本文件定義的規則在本層具有最高適用優先級。

依賴：IAlarmReporter、IAlarmProvider、IAlarmController、AlarmInfo、AlarmLevel、AlarmState、ILogger、LogLevel、ISystemStatusProvider、SystemStatus、WorkOrder

---

## 一、AlarmLevel 業務回應策略

業務流程中觸發警報時，MUST 依照 `AlarmLevel` 採取對應的業務行為：

| AlarmLevel | 業務行為 |
|------------|---------|
| `Info` | 記錄資訊性通知；MUST NOT 中斷任何進行中的業務操作 |
| `Warning` | 記錄非致命警告；作業 MUST 繼續執行；操作員 SHOULD 被通知，但無需立即介入 |
| `Error` | 作業 MUST 暫停，不得接受新的推論請求，直至對應警報被清除；正在進行中操作的中止語意（軟暫停 vs. 立即中止）由 `flows/AlarmHandling.md` 進一步定義 |
| `Critical` | 系統 MUST 停止運作，MUST NOT 接受任何新的業務操作；需人工干預後方可嘗試恢復 |

約束：

- 業務層 MUST 在收到警報清除通知（透過 `IAlarmProvider.OnAlarmChanged`，且狀態轉為 `Cleared`）後，方可恢復受 `Error` 層級警報暫停的作業

---

## 二、硬體連線中斷警報觸發規則

已配置的硬體裝置連線意外中斷時，業務層 MUST 透過 `IAlarmReporter.Raise` 建立警報。AlarmLevel 依裝置類型判定如下：

| 裝置類型 | 觸發事件 | AlarmLevel |
|---------|---------|-----------|
| 相機（`ICamera`） | `OnConnectionLost` | `Error` |
| 光源控制器（`ILightingController`） | `OnConnectionLost` | `Error` |
| 外部通訊通道（`ICommunicationChannel`） | `OnConnectionLost` | `Warning` |

約束：

- 每次連線中斷事件 MUST 產生一筆獨立的 `AlarmInfo`，其 `AlarmId` MUST 可唯一識別中斷的具體裝置實例
- 若同一裝置的警報已存在且狀態為 `Active`，MUST NOT 重複呼叫 `IAlarmReporter.Raise` 建立相同 `AlarmId` 的警報
- 觸發警報的同時，MUST 依第三節日誌規則寫入對應層級日誌

---

## 三、日誌寫入規則

以下業務事件 MUST 透過 `ILogger` 寫入日誌：

| 事件類別 | LogLevel | 說明 |
|---------|---------|------|
| 系統啟動開始 | `Info` | 啟動序列進入時 |
| 系統啟動完成（就緒） | `Info` | 系統進入就緒狀態時 |
| 系統關閉開始 | `Info` | 關閉序列進入時 |
| 系統關閉完成 | `Info` | 所有資源釋放完成後 |
| 專案載入成功 | `Info` | `IProjectProvider.Load` 回傳成功時，MUST 記錄專案名稱 |
| 專案載入失敗 | `Error` | `IProjectProvider.Load` 回傳失敗時，MUST 記錄對應 `ProjectError` 代碼 |
| 硬體裝置連線成功 | `Info` | 相機、光源控制器或外部通訊通道 `Connect` 成功時，MUST 記錄裝置識別資訊 |
| 硬體裝置連線失敗（啟動時） | `Error` | `Connect` 或 `Initialize` 回傳失敗時，MUST 記錄裝置識別與錯誤代碼 |
| 硬體裝置連線意外中斷（執行期間） | `Error` | 收到 `OnConnectionLost` 事件時，MUST 記錄裝置識別與錯誤代碼 |
| 推論模型載入成功 | `Info` | `IInspector.LoadModel` 回傳成功時 |
| 推論模型載入失敗 | `Error` | `IInspector.LoadModel` 回傳失敗時，MUST 記錄對應 `InspectionError` 代碼 |
| 警報觸發（`Info` / `Warning` 層級） | `Info` / `Warning` | 與 `AlarmLevel` 對應 |
| 警報觸發（`Error` / `Critical` 層級） | `Error` | 兩者均寫入 `Error` 層級日誌 |
| 警報清除 | `Info` | 操作員清除警報後 |

其他流程事件的日誌要求由各流程文件個別定義，MUST NOT 與本表衝突。

---

## 四、工單語意

工單（WorkOrder）是本專案業務層的核心作業識別單元。

**定義**：一個工單代表一段有明確起訖邊界的作業期間。工單識別碼（`WorkOrderId`）作為該期間所有推論結果的歸屬標記，並作為統計查詢的最小業務單位。

**業務生命週期**：

| 階段 | 說明 |
|------|------|
| 建立 | 工單識別碼確定，作業尚未開始 |
| 進行中 | 作業期間：系統接受推論請求，所有推論結果歸屬於此工單 |
| 結束 | 作業期間關閉：不再接受歸屬於此工單的新推論請求 |

約束：

- 系統 MUST 在工單進行中期間，將所有推論結果與 `WorkOrderId` 關聯後寫入
- 系統 MUST NOT 在工單結束後，繼續將新的推論結果歸屬於已結束的工單
- 工單識別碼的產生與建立觸發條件由 `flows/WorkOrder.md` 定義
- `WorkOrderId` MUST NOT 為空字串（依 Core Contract WorkOrder 約束）

---

## 五、SystemStatus 與業務流程關聯

業務流程 MUST 依 `ISystemStatusProvider.GetStatus` 的回傳值調整可執行的操作：

| SystemStatus | 業務流程限制 |
|-------------|------------|
| `Normal` | 無附加限制；所有業務操作均可依各自前置條件執行 |
| `Warning` | 作業 MUST 繼續執行；操作員 SHOULD 收到通知 |
| `Error` | 新的推論請求 MUST 拒絕執行；業務回應語意同 `AlarmLevel.Error`（見第一節） |

約束：

- 業務流程 MUST NOT 直接計算或修改 `SystemStatus`；MUST 透過 `ISystemStatusProvider.GetStatus` 查詢
- 收到 `ISystemStatusProvider.OnStatusChanged` 通知時，業務層 MUST 依新狀態值調整當下可用的業務操作

---

## 六、邊界規則

| 責任 | 所屬層 |
|------|-------|
| AlarmLevel 業務回應策略（本文件） | Flow Spec（本層） |
| `Error` 層級警報的暫停細節（軟/硬暫停） | `flows/AlarmHandling.md` |
| AlarmInfo 資料格式（欄位結構） | Core Contract |
| 警報狀態轉移機制（Active → Cleared） | Core Contract（IAlarmController） |
| SystemStatus 聚合算法（如何由子系統狀態推算整體狀態） | Core Contract（ISystemStatusProvider 實作層） |
| 日誌存檔位置、格式與保留策略 | Core Contract（ILogger 實作層） |
| 日誌的具體訊息文案 | Project Implementation |
| 工單識別碼的產生機制與觸發條件 | `flows/WorkOrder.md`（業務觸發）；Project Implementation（識別碼產生） |
