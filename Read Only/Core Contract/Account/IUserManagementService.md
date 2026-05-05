# IUserManagementService

**定位：語言無關 × 專案無關**

本文件定義平台使用者管理介面，提供帳號清單查詢、建立、角色調整、密碼更新與刪除能力。

依賴：引用 Account/UserAccount、Account/AccountOperationResult、Common/UserRole

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `ListUsers` | 無 | `UserAccount` 列表 | 回傳所有可管理的使用者帳號清單 |
| `CreateUser` | 帳號名稱（字串）、密碼（字串）、角色（`UserRole`） | `AccountOperationResult` | 建立新的使用者帳號；成功時 `Success` MUST 為 `true`，失敗時 MUST 回傳可識別錯因 |
| `UpdateUserRole` | 帳號名稱（字串）、角色（`UserRole`） | `AccountOperationResult` | 調整指定帳號的角色；成功時 `Success` MUST 為 `true`，失敗時 MUST 回傳可識別錯因 |
| `UpdatePassword` | 帳號名稱（字串）、新密碼（字串） | `AccountOperationResult` | 更新指定帳號的密碼；成功時 `Success` MUST 為 `true`，失敗時 MUST 回傳可識別錯因 |
| `DeleteUser` | 帳號名稱（字串） | `AccountOperationResult` | 刪除指定帳號；成功時 `Success` MUST 為 `true`，失敗時 MUST 回傳可識別錯因 |

## 約束

- `ListUsers` 回傳的每筆資料 MUST 包含帳號名稱與角色
- `CreateUser` 的帳號名稱 MUST NOT 為空字串
- `CreateUser` 與 `UpdatePassword` 的密碼輸入 MUST NOT 為空字串
- `CreateUser` 失敗時，`Error` MUST 為 `AccountError.DuplicateAccount` 或 `AccountError.InvalidRole`
- `UpdateUserRole` 失敗時，`Error` MUST 為 `AccountError.UserNotFound` 或 `AccountError.InvalidRole`
- `UpdatePassword` 失敗時，`Error` MUST 為 `AccountError.UserNotFound`
- `DeleteUser` 失敗時，`Error` MUST 為 `AccountError.UserNotFound`
- `CreateUser` 與 `UpdateUserRole` 失敗時，既有帳號資料 MUST NOT 被修改
- 本介面 MUST 定義平台通用的帳號管理能力，MUST NOT 描述誰可以執行這些操作的授權規則
