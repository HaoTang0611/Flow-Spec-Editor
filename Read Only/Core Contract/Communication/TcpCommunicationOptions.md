# TcpCommunicationOptions

**定位：語言無關 × 專案無關**

本文件定義 TCP 通訊協定特定配置選項，用於 `CommunicationOptions.ProtocolOptions`（`ProtocolType = Tcp` 時）。

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `RemoteAddress` | 字串 | 遠端裝置的 IP 位址或主機名稱 |
| `RemotePort` | 整數 | 遠端裝置的連接埠號 |

## 約束

- `RemoteAddress` MUST NOT 為空字串
- `RemotePort` MUST 在 `[1, 65535]` 範圍內
- `RemoteAddress` 的格式合法性（有效 IP 或主機名稱）由實作層驗證，本層僅定義語意
