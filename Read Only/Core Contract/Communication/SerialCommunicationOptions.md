# SerialCommunicationOptions

**定位：語言無關 × 專案無關**

本文件定義序列埠通訊協定特定配置選項，用於 `CommunicationOptions.ProtocolOptions`（`ProtocolType = Serial` 時）。

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `PortName` | 字串 | 序列埠識別名稱；具體格式由實作層定義 |
| `BaudRate` | 整數 | 鮑率（bps） |
| `DataBits` | 整數 | 資料位元數 |
| `StopBits` | `SerialStopBits` | 停止位元 |
| `Parity` | `SerialParity` | 同位元檢查方式 |

## SerialStopBits 枚舉

| 值 | 說明 |
|----|------|
| `One` | 1 個停止位元 |
| `Two` | 2 個停止位元 |

## SerialParity 枚舉

| 值 | 說明 |
|----|------|
| `None` | 無同位元 |
| `Even` | 偶同位元 |
| `Odd` | 奇同位元 |

## 約束

- `PortName` MUST NOT 為空字串
- `BaudRate` MUST 大於 0
- `DataBits` MUST 為 5、6、7 或 8 之一
- 具體設備支援的鮑率清單由實作層定義，本層不列舉
