# StorageConfiguration

**定位：語言無關 × 專案無關**

本文件定義專案影像存檔策略配置資料模型，描述在不同判定結果下原始影像的保留行為。

---

## 欄位

| 欄位名稱 | 型別 | 說明 |
|---------|------|------|
| `SaveRawImageOnOK` | 布林 | 判定結果為 OK 時是否保留原始影像 |
| `SaveRawImageOnNG` | 布林 | 判定結果為 NG 時是否保留原始影像 |

## 約束

- `SaveRawImageOnOK` 與 `SaveRawImageOnNG` MUST 各自獨立設定，互不影響
- `SaveRawImageOnOK` 為 `true` 時，OK 結果 MUST 保留原始影像；為 `false` 時 MUST NOT 保留
- `SaveRawImageOnNG` 為 `true` 時，NG 結果 MUST 保留原始影像；為 `false` 時 MUST NOT 保留
