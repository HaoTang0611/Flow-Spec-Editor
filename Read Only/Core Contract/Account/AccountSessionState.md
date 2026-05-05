# AccountSessionState

**定位：語言無關 × 專案無關**

本文件定義平台目前帳號登入狀態枚舉。

---

## 枚舉值

| 值 | 說明 |
|----|------|
| `Unauthenticated` | 目前沒有任何已登入帳號 |
| `Authenticated` | 目前已有帳號登入 |

## 約束

- `AccountSessionState` MUST 為上述兩個值之一，不接受其他值
