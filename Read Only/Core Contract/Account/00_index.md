# Account — 索引

**定位：語言無關 × 專案無關**

本模組定義平台帳號登入狀態、目前使用者資訊，以及使用者管理能力契約。

依賴：引用 Common/UserRole

---

## 文件清單

| 文件 | 說明 |
|------|------|
| [IAccountSession.md](IAccountSession.md) | 帳號登入狀態與登入登出能力介面 |
| [IUserManagementService.md](IUserManagementService.md) | 使用者清單與帳號管理介面 |
| [UserAccount.md](UserAccount.md) | 使用者帳號資料模型 |
| [AccountSessionState.md](AccountSessionState.md) | 帳號登入狀態枚舉 |
| [AccountError.md](AccountError.md) | 帳號認證與管理錯誤語意 |
| [AccountOperationResult.md](AccountOperationResult.md) | 帳號操作回傳結果資料模型 |
| [PlatformAction.md](PlatformAction.md) | 平台操作動作枚舉 |
| [IPermissionChecker.md](IPermissionChecker.md) | 平台權限查詢介面 |
