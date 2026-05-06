# Flow Spec — 歷史結果查詢流程

**定位：語言無關 × 專案相關**

本文件定義歷史檢測結果查詢與統計彙整的業務操作條件與執行序列。

依賴：`IResultRepository`、`QueryCondition`、`InspectionStatistics`、`DefectStatistics`、`WorkOrder`、`IPermissionChecker`、`PlatformAction`

---

## 一、流程定位

結果查詢流程提供操作員對已保存 `InspectionResult` 紀錄進行條件篩選、分頁列覽、單筆詳情讀取、統計彙整與工單清單列舉的能力。

所有查詢操作均為唯讀，MUST NOT 修改任何已保存的結果紀錄或系統狀態。

查詢操作不屬於推論請求，因此：

- 不依賴當前工單狀態（`Idle` 與 `WorkOrderActive` 均可執行）
- 不受 `cross-cutting-policy.md` 中 `AlarmLevel.Error` 暫停推論的限制

---

## 二、執行前置條件

| 前置條件 | 說明 |
|---------|------|
| 權限驗證 | 執行任何查詢操作前 MUST 確認 `IPermissionChecker.IsAllowed(GetCurrentRole(), ViewStatus)` 回傳 `true`；驗證失敗時 MUST 拒絕執行，MUST NOT 存取 `IResultRepository` |
| 業務狀態 | 查詢操作不依賴工單狀態；`Idle` 與 `WorkOrderActive` 均可執行 |

> 依 `flows/AccountSession.md` 本專案授權政策，`Guest` 及以上角色均具備 `ViewStatus` 權限，因此無論登入狀態為何，均可執行結果查詢。

---

## 三、查詢操作序列

### 3-a 條件篩選與分頁列覽

執行序列：

1. 建立 `QueryCondition`，填入所需篩選條件與分頁參數
2. 若 `From` 與 `To` 同時有值，業務層 MUST 確認 `From` 不晚於 `To`；違反時 MUST 拒絕執行並向操作員回報條件錯誤，MUST NOT 將無效條件傳入 `IResultRepository`
3. SHOULD 先呼叫 `IResultRepository.Count(condition)` 取得符合條件的總筆數，再決定分頁策略，避免請求不存在的頁面
4. 呼叫 `IResultRepository.Query(condition)`，取得 `InspectionResult` 列表

### 3-b 單筆結果詳情讀取

操作員選取特定結果識別碼後：

1. 呼叫 `IResultRepository.GetById(resultId)`
2. 若回傳空值，MUST 向操作員回報查無結果，MUST NOT 視為系統錯誤
3. 若需要關聯影像，呼叫 `IResultRepository.GetImageById(resultId)`
4. 若 `GetImageById` 回傳空值，MUST 向操作員顯示無影像狀態，MUST NOT 視為系統錯誤

### 3-c 統計資訊查詢

操作員查詢指定條件下的統計彙整：

1. 建立 `QueryCondition`（`PageIndex` 與 `PageSize` 的設定對統計計算無影響）
2. 呼叫 `IResultRepository.GetStatistics(condition)`，取得 `InspectionStatistics`
3. 無符合資料時，MUST 向操作員顯示空統計狀態，MUST NOT 視為錯誤

### 3-d 工單清單列舉

操作員查詢可供篩選使用的工單清單：

1. 呼叫 `IResultRepository.ListWorkOrders()`
2. 取得 `WorkOrder` 列表
3. 列表為空時，MUST 視為正常查詢結果，MUST NOT 視為錯誤

---

## 四、錯誤流程

| 情況 | 業務行為 |
|------|---------|
| 查詢條件無效（`From` 晚於 `To`） | 業務層 MUST 在建立 `QueryCondition` 前驗證；驗證失敗時 MUST 拒絕執行並向操作員回報條件錯誤，MUST NOT 呼叫 `IResultRepository` |
| 查詢來源不可用 | MUST 依 `IResultRepository` 回傳的空結果向操作員顯示無資料狀態，MUST NOT 視為系統錯誤 |
| 指定識別碼不存在（`GetById` 或 `GetImageById` 回傳空值） | MUST 向操作員回報查無結果或無影像，MUST NOT 視為系統錯誤 |
| 無符合查詢條件的資料 | MUST 向操作員顯示無結果狀態，MUST NOT 視為錯誤 |
| 權限驗證失敗（`ViewStatus` 未通過） | MUST 拒絕執行所有查詢操作，MUST NOT 存取 `IResultRepository`；MUST 向操作員回報無查詢權限 |

---

## 五、邊界規則

| 責任 | 所屬層 |
|------|-------|
| 查詢介面定義（`IResultRepository` 方法語意、空值語意、排序規則） | Core Contract |
| 查詢條件資料模型（`QueryCondition` 欄位語意與約束） | Core Contract |
| 統計資料模型（`InspectionStatistics`、`DefectStatistics` 欄位語意與計算規則） | Core Contract |
| 查詢操作的權限要求（`ViewStatus`）與前置驗證序列 | Flow Spec（本文件） |
| 結果查詢可在任意業務狀態執行的規定 | Flow Spec（本文件） |
| 無效條件（`From` 晚於 `To`）由業務層在呼叫 `IResultRepository` 前攔截 | Flow Spec（本文件） |
| `DefectStatistics` 跨模型加總或去重的計算規則 | Core Contract（`IResultRepository` 實作層） |
| `ListWorkOrders` 的排序方式 | Core Contract（`IResultRepository` 實作層） |
| 查詢 UI 的呈現方式（查詢表單、結果列表、圖表） | Project Implementation / UI 層 |
| 結果顯示格式（日期格式、數值精度） | Project Implementation |
| 匯出功能（CSV、報表、影像下載） | Project Implementation |
| 影像實際儲存格式與路徑 | Project Implementation（`StorageConfiguration`） |
