---
description: 解析 VSI 廠商 forecast Excel 檔（EDU、PJ 等產品線的多國家分頁格式）。當使用者上傳的 Excel 含有「Type/Segment + Buffer/Model + STD/COGS + Qty + Gross ASP」等欄位，且有多個國家代碼 (TW/KR/JP/ID/IN/VN/PH/BD+/TH/MY/SG/AZ) 分頁時自動觸發。
---

# Parse VSI Forecast File

## 觸發條件

使用者上傳的 Excel 符合以下任一特徵時觸發：

- 檔名含 `VSI_EDU`、`VSI_PJ` 或產品線縮寫
- 多個分頁名稱為國家代碼 (TW、KR、JP、ID、IN、VN、PH、BD+、TH、MY、SG、AZ)
- 國家分頁含 `Type/Segment + Model + Qty + Gross ASP` 欄位
- 月份區塊水平展開 (每月 13~14 欄)

## 產出

Wide-format DataFrame，每列一個 (product_line, country, model)，欄位：

```
product_line, country, model,
M_qty, M_gross_asp,
M+1_qty, M+1_gross_asp,
M+2_qty, M+2_gross_asp
```

`gross_asp` 依國家自動選 USD 或 Local：JP 用 Local (JPY)，其他用 USD。

## 如何使用

呼叫 `${CLAUDE_PLUGIN_ROOT}/scripts/reporter.py`。不要自己重寫解析邏輯——所有版面差異都寫在 `config/product_lines/*.json` 裡。

要單獨解析（不跑完整 report），可從 Python 直接 import：

```python
from reporter import load_configs, detect_product_line, parse_forecast_file
```

## 常見陷阱

- **TW 分頁版面與其他國家不同**（D 欄是 `COGS` 而非 `STD`），但 TW 不送 Allocate，已被自動排除
- **PJ 國家分頁底部有統計區塊**（Size、1080P、WXGA 等列），那些不是機種要過濾，parser 已用「遇 Total 列就停」處理
- **Qty=0 視為「此月無預測」**，但仍會輸出列（因為其他月份可能有預測）
- **EDU 月份區塊間 0 欄空白，PJ 月份區塊間 1 欄空白** —— stride 不同，已在各自 config 設定
