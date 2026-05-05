# IPermissionChecker

**定位：語言無關 × 專案無關**

本文件定義平台權限查詢介面，供呼叫端依指定角色與授權政策判斷是否可執行特定操作。

依賴：引用 Common/UserRole、Account/PlatformAction

---

## 方法

| 方法名稱 | 輸入 | 回傳 | 說明 |
|---------|------|------|------|
| `IsAllowed` | `UserRole`、`PlatformAction` | 布林 | 判斷指定角色是否可執行指定動作 |
| `GetPermittedActions` | `UserRole` | 列表（`PlatformAction`） | 回傳指定角色被允許的所有動作清單 |

## 約束

- `IsAllowed` 回傳結果 MUST 與目前生效的授權政策一致
- `GetPermittedActions` 回傳的清單 MUST 包含且僅包含目前授權政策允許指定角色執行的動作
- 授權政策的來源由專案層級或平台層級決定，本介面不固定角色與動作的對應矩陣
- 兩個方法均為查詢操作，MUST NOT 依賴當前登入狀態；角色由呼叫端明確傳入
