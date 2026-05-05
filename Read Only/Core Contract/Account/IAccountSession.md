# IAccountSession

**定位：語言無關 × 專案無關**

本文件定義帳號登入狀態管理介面，提供登入、登出與目前使用者查詢能力。

依賴：引用 Account/UserAccount、Account/AccountSessionState、Account/AccountOperationResult、Common/UserRole

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `SignIn` | 帳號名稱（字串）、密碼（字串） | `AccountOperationResult` | 驗證帳號並建立目前登入狀態；成功時 `Success` MUST 為 `true`，失敗時 MUST 回傳可識別錯因 |
| `SignOut` | 無 | 無 | 清除目前登入狀態；尚未登入時呼叫 MUST 靜默完成 |
| `GetCurrentUser` | 無 | `UserAccount` 或 空值 | 回傳目前登入的使用者資訊；未登入時 MUST 回傳空值 |
| `GetState` | 無 | `AccountSessionState` | 回傳目前登入狀態 |
| `IsAuthenticated` | 無 | 布林 | 回傳目前是否已登入 |
| `GetCurrentRole` | 無 | `UserRole` | 回傳目前有效角色；未登入時 MUST 回傳 `Guest`，已登入時 MUST 回傳登入帳號的 `Role` |

## 約束

- `SignIn` 成功後，`Success` MUST 為 `true`，且 `GetState` MUST 回傳 `Authenticated`
- `SignIn` 成功後，`GetCurrentUser` MUST 回傳與登入帳號對應的使用者資訊
- `SignIn` 成功後，`GetCurrentRole` MUST 回傳登入帳號的 `Role`
- `SignIn` 失敗時，`Success` MUST 為 `false`，且 `Error` MUST 可識別為 `InvalidCredential` 或 `AlreadyAuthenticated`
- `SignIn` 失敗時，既有登入狀態 MUST NOT 被修改
- `SignOut` 後，`GetState` MUST 回傳 `Unauthenticated`
- `SignOut` 後，`GetCurrentUser` MUST 回傳空值
- `SignOut` 後，`GetCurrentRole` MUST 回傳 `Guest`
- 系統初始狀態下（未曾登入），`GetCurrentRole` MUST 回傳 `Guest`
- `GetCurrentRole` MUST NOT 回傳空值；任何生命週期階段皆有有效角色
- 本介面 MUST 只定義登入狀態與目前使用者查詢能力，MUST NOT 定義任何呈現方式或授權規則
