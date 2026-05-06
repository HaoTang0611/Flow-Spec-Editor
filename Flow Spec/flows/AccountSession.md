# Flow Spec — 帳號登入登出流程

**定位：語言無關 × 專案相關**

本文件定義帳號登入登出的業務狀態轉移，以及登入角色如何影響本專案業務操作的可用性。

依賴：`IAccountSession`、`IPermissionChecker`、`IUserManagementService`、`UserAccount`、`AccountSessionState`、`AccountError`、`UserRole`、`PlatformAction`

治理規則：本文件日誌寫入行為 MUST 遵守 `cross-cutting-policy.md`。

---

## 一、流程定位

帳號登入狀態決定本專案業務操作的可用範圍。`IAccountSession` 的介面語意（`SignIn`、`SignOut`、`GetCurrentRole` 等）已由 Core Contract 定義，本文件補充：

1. 本專案的角色授權政策（`UserRole` 與 `PlatformAction` 的對應關係）
2. 登入登出對進行中業務作業（工單）的影響規則
3. 帳號管理操作的業務前置條件

---

## 二、業務狀態定義

| 狀態 | 說明 |
|------|------|
| `Unauthenticated` | 無帳號登入；系統以 `Guest` 角色運作 |
| `Authenticated` | 帳號已登入；系統以登入帳號的 `UserRole` 運作 |

> 這兩個狀態對應 Core Contract `AccountSessionState` 的枚舉值，此處直接採用。

---

## 三、狀態轉移

| 前置狀態 | 觸發條件 | 後繼狀態 | 副作用 |
|---------|---------|---------|-------|
| `Unauthenticated` | `IAccountSession.SignIn` 回傳成功 | `Authenticated` | 記錄登入成功日誌（`Info`，含帳號名稱與角色） |
| `Unauthenticated` | `IAccountSession.SignIn` 回傳失敗（`InvalidCredential`） | `Unauthenticated` | 記錄登入失敗日誌（`Warning`，含帳號名稱與 `AccountError` 代碼） |
| `Authenticated` | `IAccountSession.SignIn` 回傳失敗（`AlreadyAuthenticated`） | `Authenticated`（不變） | — |
| `Authenticated` | `IAccountSession.SignOut` 呼叫 | `Unauthenticated` | 記錄登出日誌（`Info`，含帳號名稱） |

> 若需要切換至不同帳號，MUST 先呼叫 `SignOut`，再呼叫 `SignIn`；不支援在已登入狀態下直接登入其他帳號。

---

## 四、本專案授權政策

業務層執行受保護操作前，MUST 透過 `IPermissionChecker.IsAllowed(GetCurrentRole(), action)` 驗證目前角色的權限。本專案的授權政策如下：

| `UserRole` | 允許的 `PlatformAction` |
|-----------|----------------------|
| `Guest` | `ViewStatus` |
| `Operator` | `ViewStatus`、`ExecuteOperation`、`LoadProject` |
| `Engineer` | `ViewStatus`、`ExecuteOperation`、`LoadProject`、`ModifyProject` |
| `Admin` | 所有（`ViewStatus`、`ExecuteOperation`、`LoadProject`、`ModifyProject`、`ManageAccounts`） |

約束：

- 業務層執行任何受保護操作前 MUST 呼叫 `IPermissionChecker.IsAllowed`；驗證失敗（回傳 `false`）時 MUST 拒絕執行對應操作
- `Guest` 角色涵蓋未登入狀態（`Unauthenticated`）以及已登入使用者的最低限制角色
- 上述對應關係為本專案的授權政策基準；若 Project Implementation 採用可設定的授權政策設定檔，以設定檔實際內容為準，且設定結果 MUST 可透過 `IPermissionChecker.IsAllowed` 正確反映

---

## 五、登入登出對業務作業的影響

### 5-a 登入時（Unauthenticated → Authenticated）

- 若系統目前有進行中的工單（`WorkOrderActive`），登入操作 MUST NOT 中斷工單進行
- 登入成功後，有效角色立即切換為登入帳號的 `UserRole`，後續所有業務操作均依新角色判斷可用性
- 同一時間 MUST 只有一個帳號處於登入狀態（由 `IAccountSession` 合約保證）

### 5-b 登出時（Authenticated → Unauthenticated）

- 若系統目前有進行中的工單（`WorkOrderActive`），登出操作 MUST NOT 自動結束工單
- 登出後有效角色立即切換回 `Guest`；`Guest` 角色不允許 `ExecuteOperation`，因此登出後無法觸發新的推論或操作工單
- 若工單進行中因登出導致角色降為 `Guest`，工單 MUST 維持 `WorkOrderActive` 狀態；工單的結束 MUST 由具備 `ExecuteOperation` 權限的使用者重新登入後執行

---

## 六、帳號管理操作的業務約束

帳號管理操作（`CreateUser`、`UpdateUserRole`、`UpdatePassword`、`DeleteUser`）需要 `ManageAccounts` 授權（僅 `Admin` 角色）：

- 執行前 MUST 確認 `IPermissionChecker.IsAllowed(GetCurrentRole(), ManageAccounts)` 回傳 `true`
- 帳號管理操作可在系統就緒後的任意時間執行，不依賴工單狀態或警報狀態
- 操作成功後，MUST 記錄對應日誌（`Info`，含操作類型與目標帳號名稱）
- 操作失敗後，MUST 記錄日誌（`Warning`，含操作類型與 `AccountError` 代碼）

---

## 七、日誌規則（AccountSession 追加）

以下事件由本流程追加，MUST NOT 與 `cross-cutting-policy.md` 第三節衝突：

| 事件 | `LogLevel` | 說明 |
|------|-----------|------|
| 登入成功 | `Info` | MUST 記錄帳號名稱與角色 |
| 登入失敗 | `Warning` | MUST 記錄帳號名稱與 `AccountError` 代碼 |
| 登出 | `Info` | MUST 記錄帳號名稱 |
| 帳號管理操作成功 | `Info` | MUST 記錄操作類型與目標帳號名稱 |
| 帳號管理操作失敗 | `Warning` | MUST 記錄操作類型與 `AccountError` 代碼 |

---

## 八、邊界規則

| 責任 | 所屬層 |
|------|-------|
| `AccountSessionState` 轉移機制（`SignIn`、`SignOut` 的合約語意） | Core Contract（`IAccountSession`） |
| 本專案角色與 `PlatformAction` 的授權矩陣（本文件第四節） | Flow Spec（本層） |
| 授權政策的儲存格式與設定方式（是否可由設定檔覆蓋） | Project Implementation |
| 登入 UI 呈現（輸入欄位、錯誤提示）方式 | Project Implementation / UI 層 |
| 帳號名稱與密碼的格式限制（長度、字元集） | Project Implementation |
| 刪除目前登入中帳號的業務強制登出邏輯 | Project Implementation |
| 角色切換後 UI 可用性更新的呈現方式 | Project Implementation / UI 層 |
