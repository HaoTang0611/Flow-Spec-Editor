# IODirection

**定位：語言無關 × 專案無關**

本文件定義 IO 通道方向枚舉。

---

## 枚舉值

| 值 | 說明 |
|----|------|
| `Input` | 輸入通道（只讀） |
| `Output` | 輸出通道（可寫） |

## 約束

- `Direction` MUST 為 `Input` 或 `Output`，不接受其他值
