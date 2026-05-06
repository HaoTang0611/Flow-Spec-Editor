# Flow Spec — 警報處理流程

**定位：語言無關 × 專案相關**

本文件定義警報從觸發至清除的業務處理流程，以及不同 `AlarmLevel` 下業務作業的暫停、操作員清除與恢復序列。

依賴：`IAlarmReporter`、`IAlarmProvider`、`IAlarmController`、`AlarmInfo`、`AlarmLevel`、`AlarmState`、`ILogger`

治理規則：`AlarmLevel` 業務回應策略（Info/Warning/Error/Critical 的高層次行為）已由 `cross-cutting-policy.md` 第一節定義；本文件補充作業暫停細節、操作員清除序列與恢復條件。

---

## 一、流程定位

警報系統為跨流程的橫切關注點。本文件聚焦三個業務問題：

1. 業務作業在不同警報狀態下允許哪些操作（業務狀態機）
2. 操作員如何清除警報、系統如何恢復業務作業（清除序列）
3. `Critical` 層級警報的業務終止語意

`Info` 與 `Warning` 層級警報的業務回應語意已足夠清晰（不中斷作業），本文件不重複定義，僅在邊界規則中說明。

---

## 二、業務狀態定義

| 狀態 | 說明 |
|------|------|
| `Normal` | 無 `Active` 狀態的 `Error` 或 `Critical` 層級警報；所有業務操作依各自前置條件執行 |
| `ErrorSuspended` | 存在一筆或多筆 `Active` 狀態的 `Error` 層級警報；新的推論請求 MUST 被拒絕 |
| `CriticalFault` | 存在一筆或多筆 `Active` 狀態的 `Critical` 層級警報；系統 MUST 停止所有業務操作 |

> `Info` 與 `Warning` 層級警報不改變業務狀態；系統在這兩個層級的 `Active` 警報存在時維持 `Normal` 狀態。

---

## 三、狀態轉移

| 前置狀態 | 觸發條件 | 後繼狀態 | 副作用 |
|---------|---------|---------|-------|
| `Normal` | `IAlarmReporter.Raise` 成功建立 `Error` 層級警報 | `ErrorSuspended` | 依 `cross-cutting-policy.md` 第三節寫入 `Error` 日誌 |
| `Normal` | `IAlarmReporter.Raise` 成功建立 `Critical` 層級警報 | `CriticalFault` | 依 `cross-cutting-policy.md` 第三節寫入 `Error` 日誌 |
| `ErrorSuspended` | `IAlarmReporter.Raise` 成功建立 `Critical` 層級警報 | `CriticalFault` | 依 `cross-cutting-policy.md` 第三節寫入 `Error` 日誌 |
| `ErrorSuspended` | 所有 `Active` 狀態的 `Error` 層級警報均已轉為 `Cleared` | `Normal` | 記錄 `Info` 日誌（業務作業已恢復可用）；業務層 MUST 開放後續推論請求 |
| `CriticalFault` | 系統重新啟動（見 `flows/Startup.md`） | `Normal`（依啟動結果） | 重啟後由啟動流程重新建立有效狀態 |

---

## 四、ErrorSuspended 期間的業務約束

`ErrorSuspended` 期間，業務層 MUST 遵守：

- 新的即時推論請求（`IInspector.Inspect`）MUST 被拒絕，MUST NOT 進入取像或推論序列
- 新的批次推論請求（`IBatchInspector` 啟動）MUST 被拒絕
- 工單建立 MUST 被拒絕

恢復條件：

- 業務層 MUST 訂閱 `IAlarmProvider.OnAlarmChanged`，在收到 `Cleared` 狀態通知後評估是否可恢復
- 恢復前 MUST 透過 `IAlarmProvider.ListByState(Active)` 確認已無任何 `Error` 層級 `Active` 警報；確認後方可轉移至 `Normal`

---

## 五、CriticalFault 期間的業務約束

`CriticalFault` 期間：

- 系統 MUST 停止接受所有新的業務操作
- MUST NOT 嘗試透過清除 `Critical` 層級警報來恢復業務作業；`CriticalFault` 狀態唯一的退出路徑為系統重新啟動
- 重新啟動後，系統 MUST 依 `flows/Startup.md` 重新執行完整初始化序列

---

## 六、操作員清除警報序列

業務層清除 `Error` 層級警報並恢復作業的完整序列：

1. 操作員確認對應警報的觸發條件已在硬體或環境層面排除
2. 操作員對目標警報識別碼呼叫 `IAlarmController.Clear`
3. `IAlarmProvider.OnAlarmChanged` 觸發（目標警報狀態更新為 `Cleared`）；MUST 記錄警報清除日誌（`Info`，含警報識別碼）
4. 業務層評估是否仍有其他 `Active` 狀態的 `Error` 層級警報：
   - 若無：系統轉移至 `Normal`，記錄 `Info` 日誌（業務作業恢復可用）
   - 若有：維持 `ErrorSuspended`，等待剩餘警報被逐一清除

約束：

- `IAlarmController.Clear` 的輸入 MUST 為合法的警報識別碼（MUST NOT 為空字串）
- 目標警報識別碼不存在時，`Clear` 回傳 `false`；業務層 MUST 視為該警報已不存在，不影響業務狀態
- 目標警報已為 `Cleared` 時，`Clear` 回傳 `true`；業務層 MUST 按正常清除流程繼續評估剩餘警報

---

## 七、Info 與 Warning 層級警報的業務處理

- `Info` 層級警報：MUST NOT 限制任何業務操作；系統維持 `Normal` 狀態；操作員可隨時清除
- `Warning` 層級警報：業務作業 MUST 繼續執行；操作員應收到通知，但無需立即介入；操作員可隨時清除

這兩個層級的 `Active` 警報存在或被清除均 MUST NOT 改變業務狀態（不觸發狀態轉移）。

---

## 八、邊界規則

| 責任 | 所屬層 |
|------|-------|
| `AlarmLevel` 業務回應策略（分層行為規則） | `cross-cutting-policy.md`（本層） |
| 硬體斷線觸發警報的時機與 `AlarmLevel` | `cross-cutting-policy.md`（本層） |
| `Error` 層級暫停期間正在進行中操作的中止語意（軟/硬暫停） | Project Implementation |
| `AlarmInfo` 資料格式（`AlarmId`、`Level`、`State`、`Code`、`Message`） | Core Contract |
| 警報識別碼的產生規則與具體 `AlarmCode` 的定義 | Project Implementation |
| 操作員清除警報的 UI 入口與步驟確認流程 | Project Implementation / UI 層 |
| `Critical` 層級警報後系統重啟的觸發方式（按鈕、指令） | Project Implementation / UI 層 |
| `AlarmState` 的轉移機制（`Active` → `Cleared`） | Core Contract（`IAlarmController`） |
| 警報清除通知機制（`OnAlarmChanged`） | Core Contract（`IAlarmProvider`） |
