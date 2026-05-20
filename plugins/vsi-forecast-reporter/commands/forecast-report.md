---
description: 整理 VSI 廠商 forecast 檔（EDU/PJ 等多產品線）為標準 M/M+1/M+2 大表，比對三份 Allocate Table（Feb/Mar/Apr 等）找出缺漏機種與缺漏 ASP，並把 Master 資料回填到 Allocate Table 產出可直接上傳的版本。一次產出 6 份檔案：Master + 2 份問卷 + 3 份填好的 Allocate。
---

# /forecast:report — 一鍵產出大表、Missing 報告與填好的 Allocate Table

## 這個指令做什麼

把使用者上傳的「廠商 forecast 檔（一個或多個）」與「三份 Allocate Table（M/M+1/M+2）」整合處理，產出 6 份檔案：

| # | 檔名 | 用途 | 給誰 |
|---|---|---|---|
| 1 | `master_forecast.xlsx` | M / M+1 / M+2 三月份大表 | 內部參考 |
| 2 | `missing_models_for_BLM.xlsx` | forecast 有 Qty 但 Allocate 沒此列 | BLM 審查 |
| 3 | `missing_asp_for_CSC.xlsx` | 有 Qty 但 Gross ASP 為空的機種 | CSC 詢價 |
| 4 | `{原檔名}_filled.xlsx` (M) | 填好的 Allocate Table | 上傳系統 |
| 5 | `{原檔名}_filled.xlsx` (M+1) | 同上 | 同上 |
| 6 | `{原檔名}_filled.xlsx` (M+2) | 同上 | 同上 |

## 使用者會怎麼說

- "/forecast:report 把這幾份 forecast 跟三份 Allocate Table 跑一下"
- "幫我整理 EDU 跟 PJ 的 forecast"
- 或直接 "/forecast:report" 後接著上傳檔案

## 執行步驟

### Step 1: 確認檔案

從對話中找出：

- **Forecast 檔案** (一個或多個，必要)：通常檔名含 `VSI_EDU`、`VSI_PJ` 等產品線代號
- **Allocate Table 檔案** (剛好三個，必要)：通常檔名含 `YYYY_M_xxx_AllocateTable.xlsx` 格式

如果缺少其中一類、或 Allocate 不是三份，先問使用者再執行。

### Step 2: 執行核心引擎

```bash
python ${CLAUDE_PLUGIN_ROOT}/scripts/reporter.py \
    --forecasts <forecast1.xlsx> <forecast2.xlsx> ... \
    --allocates <allocate1.xlsx> <allocate2.xlsx> <allocate3.xlsx> \
    --config-dir ${CLAUDE_PLUGIN_ROOT}/config \
    --output-dir /mnt/user-data/outputs
```

- Allocate 檔案順序不重要，Plugin 會從檔名自動辨識 (year, month) 並排序
- forecast 也不必依產品線順序排列

### Step 3: 解讀結果

腳本 stderr 印出五個階段的摘要：

```
[1/5] 載入設定        ← 顯示產品線、Local 幣別國家
[2/5] 解析 forecast    ← 顯示各檔案、各國家分頁的機種數
[3/5] 比對 Allocate    ← 確認三份 Allocate 對應到 M/M+1/M+2
[4/5] 輸出 Master + Missing 報告
[5/5] 回填 Allocate    ← 顯示各月份填了幾列
```

重點數字：

- **已填 N 列**：成功對應的列數
- **Master 未提到 N 列**：Allocate 上但 forecast 沒提到（其他產品線負責或真的無預測）
- **Missing Models**：要送 BLM 審查
- **Missing ASP**：要送 CSC 詢價

### Step 4: 呈現給使用者

用 `present_files` 提供全部 6 份檔案，並用 2-3 句話總結：

- 大表共幾列
- 各月份填好幾列 Allocate
- 有幾個 Missing Model / Missing ASP 警告
- 預期下一步：把 missing 兩份檔送出、填好的 Allocate Table 上傳到 Allocation 系統

## 重要規則

- **絕對不要自己重寫 reporter.py 的邏輯**，所有規則寫在 `config/` 裡
- **不要修改原始 Allocate Table**，產出版本一律加 `_filled` 後綴
- 預設 **以 Master 為準完全覆寫** Allocate Table（這是使用者明確要求）
- Allocate Table 的「`Forecase Qty`」是原始拼字（少了 t），不要寫成 `Forecast Qty`
- TW 國家自動跳過
- JP 國家用 Local 幣別 (JPY)，其他國家用 USD

## 新增產品線怎麼辦

如果使用者上傳了不認識的產品線檔案（例如 LCD），先告訴使用者：

> 「我沒辨識出 `{filename}` 是哪個產品線。如果這是新產品線，需要在 `config/product_lines/` 新增一個 JSON 設定檔，描述：
> - filename_patterns（檔名特徵）
> - country_sheets（國家分頁清單）
> - country_layout（header_row、cols_per_month 等）
> 
> 可以參考 `EDU.json` 或 `PJ.json` 的格式。需要我幫你產生新的 config 嗎？」

## 失敗處理

| 問題 | 對策 |
|---|---|
| Allocate Table 數量不是 3 份 | 詢問使用者是否要用更少 / 更多月份 |
| 檔名抓不到 year_month | 提示使用者重新命名為 `YYYY_M_*` 格式 |
| 找不到 forecast 的產品線 | 列出已支援的產品線，請使用者確認或新增 config |
| 0 列被填入 | 通常是 forecast 月份跟 Allocate 對不上，要檢查 |
