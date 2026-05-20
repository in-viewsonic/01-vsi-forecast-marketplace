# VSI Forecast Reporter

整理多產品線廠商 forecast 檔，產出 Master 大表、Missing 報告，並回填 Allocate Table。

## 解決什麼問題

每個月廠商（EDU / PJ / 未來其他產品線）會交來各自的 6M 滾動預測 Excel，格式不一：

- EDU 用「Type + Buffer + Model」三欄，每月 13 欄無間隔
- PJ 用「Segment + Model」兩欄，每月 13+1 欄
- TW 分頁是 COGS、其他是 STD
- 國家分頁底部還有 segment 統計區塊干擾解析

公司 Allocate 系統要的格式又是另一回事：

- 預先列好 14,000+ 列 (Country × Model)
- 只要填兩個值：`Model Price` 和 `Forecase Qty`
- 三個月份各一份檔（Feb/Mar/Apr ...）

人工處理光是攤平加比對就要好幾天。這個 plugin 把整個流程自動化。

## 它做什麼

執行 `/forecast:report` 後，一次產出 **6 份檔案**：

| 檔案 | 用途 |
|---|---|
| `master_forecast.xlsx` | 攤平的 M/M+1/M+2 大表 |
| `missing_models_for_BLM.xlsx` | 有 Qty 但 Allocate 沒此機種 → 送 BLM 審 |
| `missing_asp_for_CSC.xlsx` | 有 Qty 但缺 Price → 送 CSC 詢價 |
| `{原檔名}_filled.xlsx` × 3 | 三份填好的 Allocate Table，可直接上傳 Allocation 系統 |

## 安裝

### 方式 A：本地測試

```bash
git clone <this-repo>
# 在 Claude Code 內：
/plugin marketplace add ./vsi-forecast-marketplace
/plugin install vsi-forecast-reporter@vsi-tools
```

### 方式 B：從遠端 Marketplace

```
/plugin marketplace add <團隊內部 git URL>
/plugin install vsi-forecast-reporter@vsi-tools
```

## 用法

### 主流程：一鍵產出全部

```
/forecast:report
```

接著上傳：
- 一或多份 forecast 檔（VSI_EDU_*.xlsx、VSI_PJ_*.xlsx 等）
- 三份 Allocate Table（檔名 `YYYY_M_*_AllocateTable.xlsx`）

順序不重要，plugin 會自動辨識。

### 輔助：只檢查不執行

```
/forecast:inspect
```

只解析 forecast、顯示結構摘要，不修改任何檔案。用來除錯或驗證新月份的檔案。

## 怎麼新增產品線

不用改程式碼，只要在 `config/product_lines/` 加一個 JSON。

### Step 1: 觀察新產品線檔案

用 `/forecast:inspect` 跑跑看，記下：

- 國家分頁清單
- 欄位標題在第幾列
- 資料從第幾列開始
- 每月幾欄、區塊間有沒有空欄
- Model 欄是哪一欄、Qty / Gross ASP USD / Gross ASP Local 各自的 offset

### Step 2: 複製 EDU.json 改寫

```json
{
  "product_line": "LCD",
  "filename_patterns": ["VSI[_-]?LCD"],
  "content_signature": {
    "row": 12,
    "must_contain": ["Type", "Model"]
  },
  "country_sheets": ["KR", "JP", "ID", ...],
  "country_layout": {
    "header_row": 12,
    "data_start_row": 13,
    "cols_per_month": 13,
    "gap_cols_between_blocks": 0,
    "first_block_start_col": 4
  },
  "dimension_columns": {
    "model": 3
  },
  "metric_offsets": {
    "qty": 1,
    "gross_asp_usd": 3,
    "gross_asp_local": 4
  }
}
```

### Step 3: 驗證

跑 `/forecast:inspect` 確認新產品線檔案被正確辨識且各國家機種數合理。

## 怎麼新增 Local 幣別國家

只改 `config/rules.json` 一行：

```json
"local_currency_countries": ["JP", "KR", "IN"]
```

清單上的國家會用 `gross_asp_local` 欄當售價，其他用 `gross_asp_usd`。

## 結構

```
vsi-forecast-reporter/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── forecast-report.md      ← /forecast:report 主指令
│   └── forecast-inspect.md     ← /forecast:inspect 檢查
├── skills/
│   ├── parse-forecast/SKILL.md
│   ├── build-master/SKILL.md
│   └── find-missing/SKILL.md
├── scripts/
│   └── reporter.py             ← 核心引擎 (500 行)
├── config/
│   ├── rules.json              ← 通用規則
│   └── product_lines/
│       ├── EDU.json
│       └── PJ.json
└── README.md
```

## 設計原則

1. **新產品線只動 config，不改程式碼**：每個產品線一個 JSON，描述版面差異
2. **以 Master 為準回填**：完全覆寫 Allocate Table 已有值，是使用者明確要求
3. **原檔不動**：產出版本一律加 `_filled` 後綴
4. **TW 自動跳過**：不送 Allocation 系統
5. **JP 用 JPY、其他用 USD**：由 `local_currency_countries` 控制

## 已知限制

- 假設同一個 (Country, Model) 不會在多個產品線都出現（目前 EDU/PJ 確實零重疊）
- Allocate Table 三份必須是同一個 sheet 結構（11 欄、`Forecase Qty` 在欄 11）
- 月份數固定為 3（M / M+1 / M+2），未來如要 4 個月以上要改 `rules.json` 的 `output_months`

## 維護者

VSI Team。問題請開 issue 或在 Slack #vsi-tools 詢問。
