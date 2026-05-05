# LightingBrightnessRange

**定位：語言無關 × 專案無關**

本文件定義光源頻道亮度原生單位的合法值範圍資料模型，由 `ILightingController.GetBrightnessRange` 回傳。

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `Min` | 整數 | 亮度最小值（控制器原生單位） |
| `Max` | 整數 | 亮度最大值（控制器原生單位） |

## 約束

- `Min` MUST 為非負整數
- `Max` MUST 大於 `Min`
- `Min` 與 `Max` 的物理語意（DAC 階數、瓦數等）由控制器硬體決定，Core Contract 不定義
