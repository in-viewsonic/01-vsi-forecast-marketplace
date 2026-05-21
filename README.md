# VSI Tools — Claude Code Plugin Marketplace

VSI 內部用的 Claude Code Plugin 集合。

## 已收錄的 Plugin

| Plugin | 用途 |
|---|---|
| [vsi-forecast-reporter](./plugins/vsi-forecast-reporter) | 廠商 forecast 整理 + Allocate Table 自動填寫 |

（未來會加更多）

## 怎麼安裝

### Step 1: 加入 Marketplace

在 Claude Code 內執行：

```
/plugin marketplace add <團隊 git URL 或本地路徑>
```

例如：
- 本地：`/plugin marketplace add ./vsi-forecast-marketplace`
GitHub：`/plugin marketplace add github.com/in-viewsonic/01-vsi-forecast-marketplace`

### Step 2: 安裝想用的 Plugin

```
/plugin install vsi-forecast-reporter@vsi-tools
```

### Step 3: 使用

例如執行 forecast 整理：

```
/forecast:report
```

接著上傳 forecast 跟 Allocate Table 檔案。

## 更新 Plugin

如果有新版本：

```
/plugin marketplace update vsi-tools
```

## 開發者：怎麼新增 Plugin 到這個 Marketplace

1. 把新 Plugin 放到 `plugins/<plugin-name>/`
2. 在 `.claude-plugin/marketplace.json` 的 `plugins` 陣列新增一筆
3. Commit + push
4. 使用者執行 `/plugin marketplace update vsi-tools` 就能看到新 plugin

## 注意事項

- Marketplace 名稱 `vsi-tools` 是這個目錄的識別，不能取跟 Anthropic 保留字衝突的名稱（例如 `anthropic-*`、`claude-*`）
- 每個 plugin 的 `version` 欄要正確標記，使用者只有在版本改變時才會收到更新通知
- Plugin 的程式碼/設定都在各自的 `plugins/<name>/` 目錄底下管理
