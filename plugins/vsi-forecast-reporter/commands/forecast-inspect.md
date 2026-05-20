---
description: 檢查 forecast 檔的結構而不執行完整流程。用來在跑 /forecast:report 之前確認檔案能被正確辨識、每國家解析到幾個機種、有沒有解析異常。也可以驗證新增的產品線 config 是否寫對。
---

# /forecast:inspect — 檢查 forecast 檔結構（dry-run）

## 這個指令做什麼

不修改任何檔案，只執行解析並回報摘要：

- 每個 forecast 檔被辨識為哪個產品線
- 每個國家分頁解析到幾個機種
- 哪些國家分頁找不到、或解析 0 列（可能 config 寫錯）

## 什麼時候用

- 新月份的 forecast 拿到後，先確認結構沒變
- 新增產品線 config 後驗證寫對沒
- `/forecast:report` 結果怪怪的，要追蹤是哪個分頁出問題

## 執行步驟

呼叫 reporter.py 但**不傳 `--allocates`**，並且**不要 present_files**，把結果整理成 chat 訊息即可。

由於 reporter.py 目前要求 `--allocates`，inspect 模式建議改用 Python 直接呼叫解析函式：

```python
import sys
sys.path.insert(0, "${CLAUDE_PLUGIN_ROOT}/scripts")
from reporter import load_configs, detect_product_line, parse_forecast_file
from pathlib import Path

config_dir = Path("${CLAUDE_PLUGIN_ROOT}/config")
pl_configs, rules = load_configs(config_dir)
num_months = rules["output_months"]
local_currency = rules.get("local_currency_countries", [])

for fpath in [Path(f) for f in forecast_files]:
    cfg = detect_product_line(fpath, pl_configs)
    print(f"{fpath.name} → {cfg['product_line'] if cfg else '未辨識'}")
    if cfg:
        df = parse_forecast_file(fpath, cfg, num_months, local_currency)
        # 用 country 分組統計
        print(df.groupby('country').size())
```

## 輸出格式建議

用 Markdown 表格呈現：

```
| 產品線 | 國家 | 解析機種數 | 有 M 月 Qty | 有 M+1 月 Qty | 有 M+2 月 Qty |
|---|---|---|---|---|---|
| EDU | AZ | 14 | 4 | 9 | 7 |
| EDU | PH | 27 | 14 | 13 | 11 |
```

並提醒使用者：

- 全 0 的國家分頁可能 layout config 錯了
- 列數差距很大的相鄰月份可能該分頁有缺漏

## 不做什麼

- 不寫任何檔案到 outputs
- 不呼叫 present_files
- 不比對 Allocate Table（那是 /forecast:report 才要做的）
