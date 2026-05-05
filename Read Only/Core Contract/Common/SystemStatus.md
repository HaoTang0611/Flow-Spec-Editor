# SystemStatus

**定位：語言無關 × 專案無關**

本文件定義平台系統整體狀態枚舉。

---

## 枚舉值

| 值 | 說明 |
|----|------|
| `Normal` | 系統運作正常，無異常 |
| `Warning` | 系統存在非致命異常，仍可運作 |
| `Error` | 系統發生致命錯誤，無法正常運作 |

## 約束

- `SystemStatus` 為平台整體健康狀態的語意描述，MUST 由平台層根據各子系統狀態綜合決定
- `SystemStatus` MUST NOT 與單一模組的錯誤狀態混用
- 三個值具有嚴格的嚴重度排序：`Normal` < `Warning` < `Error`；實作 MUST NOT 將較嚴重的狀態回報為較輕微的值
