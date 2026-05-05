# AccountError

**定位：語言無關 × 專案無關**

本文件定義帳號認證與管理操作錯誤語意。

---

## 錯誤類型

| 錯誤名稱 | 觸發條件 |
|---------|---------|
| `InvalidCredential` | `SignIn` 提供的帳號名稱或密碼無效 |
| `AlreadyAuthenticated` | 已有帳號登入時再次要求建立新的登入狀態 |
| `NotAuthenticated` | 需要目前登入帳號的操作發生在未登入狀態 |
| `UserNotFound` | 帳號管理操作指定的帳號不存在 |
| `DuplicateAccount` | 建立使用者時，帳號名稱已存在 |
| `InvalidRole` | 輸入的角色值不在 `UserRole` 定義範圍內 |

## 約束

- 上述錯誤 MUST 可在帳號認證或使用者管理操作回傳失敗時，供呼叫端識別失敗原因
