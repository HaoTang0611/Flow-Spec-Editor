# LightingState

**定位：語言無關 × 專案無關**

本文件定義光源控制器連線狀態枚舉。

---

## 枚舉值

| 值 | 說明 |
|----|------|
| `Disconnected` | 未建立連線 |
| `Connecting` | 連線建立中 |
| `Connected` | 連線正常，可控制光源頻道 |
| `Error` | 連線中斷或控制器回報硬體錯誤 |

## 約束

- `LightingState` 為光源控制器可觀察的連線狀態，MUST NOT 包含業務層的「發光中」語意
