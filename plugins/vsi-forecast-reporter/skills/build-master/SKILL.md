---
description: 把多份 forecast 檔合併攤平為單一 Master 大表，含 M、M+1、M+2 三個月份的 Qty 與 Gross ASP。當使用者要求「整理 forecast」、「合併多個 forecast 檔」、「攤平成大表」時觸發。
---

# Build Master Forecast Table

## 觸發條件

- 使用者已上傳多份 forecast 並要求合併
- 提到「大表」、「攤平」、「整理 forecast」、「跨產品線整合」
- 或上下文需要單一視圖看所有 forecast

## 大表格式 (Wide)

每個 (product_line, country, model) 一列，9 欄：

| 欄位 | 說明 |
|---|---|
| product_line | EDU / PJ / 未來新增 |
| country | KR / JP / PH / ... (TW 排除) |
| model | 機種代碼 |
| M_qty | 當月預測數量 |
| M_gross_asp | 當月售價（JP→JPY、其他→USD） |
| M+1_qty / M+1_gross_asp | 下個月 |
| M+2_qty / M+2_gross_asp | 下下個月 |

數值四捨五入到小數第 2 位。

## 設計原則

- **Wide format 而非 Long**：使用者偏好「一列看完一個機種跨三月」
- **單一 ASP 欄而非 USD/Local 雙欄**：避免下游搞混，由 country 自動決定幣別
- **空值保留 None**：不要硬填 0，避免跟「真的賣 0」混淆

## 排序

預設依 `(product_line, country, model)` 排序，方便人眼閱讀。

## 配置

「哪些國家用 Local」寫在 `config/rules.json` 的 `local_currency_countries` 欄位：

```json
"local_currency_countries": ["JP"]
```

未來新增國家直接加進這個 list，例如 `["JP", "KR"]`。
