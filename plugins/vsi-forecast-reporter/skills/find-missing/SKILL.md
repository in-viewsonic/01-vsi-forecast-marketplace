---
description: 比對 Master forecast 與 Allocate Table，找出兩類缺漏：(1) Missing Models - forecast 有 Qty 但 Allocate 沒此 (Country, Model) 列；(2) Missing ASP - 有 Qty 但 Gross ASP 為空。用於送 BLM 審查新機種、送 CSC 詢價。
---

# Find Missing Models & Missing ASP

## 觸發條件

- 已產出 Master 大表後
- 使用者問「哪些機種需要 BLM 確認」、「哪些要詢價」
- 或自動跟在 `/forecast:report` 流程中

## 兩類缺漏定義

### Missing Models (給 BLM)

對 Master 大表每一列 (country, model)：

- 該月 Qty > 0
- 但 (country, model) 不在「該月對應的 Allocate Table」中
- → 紀錄為 Missing Model

代表這些機種有預測量但 Allocate 系統還沒建立，需要 BLM 審查後新增。

### Missing ASP (給 CSC)

對 Master 大表每一列：

- 該月 Qty > 0
- 但該月 Gross ASP 為 0 或空
- → 紀錄為 Missing ASP

代表廠商給的 forecast 漏填價格，CSC 需要找 BLM/NS 詢問。

## 輸出欄位

兩份報告都用相同的基本欄位 + 各自的 reason：

```
month, product_line, country, model, qty, gross_asp, reason
```

按月份 + 國家排序，方便分批處理。

## 規則

- TW 整個排除（不送 Allocate）
- Qty=0 視為無預測，不算 Missing
- 月份對應：Allocate 檔名 `YYYY_M_*` 抓出年月，依序對應 M / M+1 / M+2

## 配置

Missing 判定規則寫在 `config/rules.json` 的 `missing_rules`：

```json
"missing_rules": {
  "treat_qty_zero_as_no_qty": true
}
```
