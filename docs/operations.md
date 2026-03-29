# 運營手冊：監控、故障恢復、部署回滾

## 一、監控和可觀測性

### 1.1 Cloudflare 內建監控

| 監控項 | 工具 | 檢查頻率 |
|---|---|---|
| Workers 錯誤率 | Cloudflare Dashboard → Workers → Analytics | 每日 |
| Workers 請求量 | Cloudflare Dashboard | 每日 |
| D1 查詢效能 | Cloudflare Dashboard → D1 | 每週 |
| Pages 部署狀態 | Cloudflare Dashboard → Pages | 每次部署 |

### 1.2 API 成本告警

在 Workers 中實作成本追蹤：

```typescript
// middleware/cost-tracker.ts
export async function trackCost(c: Context, next: Next) {
  const start = Date.now()
  await next()
  const duration = Date.now() - start

  // 記錄到 D1
  await c.env.DB.prepare(
    'INSERT INTO api_usage_log (endpoint, duration_ms, timestamp) VALUES (?, ?, ?)'
  ).bind(c.req.path, duration, new Date().toISOString()).run()
}
```

每日檢查 API 花費是否超過預算：
- 每日 AI API 預算上限：$5 USD
- 超過時自動降級到免費模型或暫停新請求

### 1.3 Agent 運營監控

透過 Paperclip API 定期檢查：

```bash
# 檢查所有 agent 狀態
curl -s http://127.0.0.1:3100/api/companies/{companyId}/agents | \
  python3 -c "import sys,json; [print(f\"{a['name']}: {a['status']}\") for a in json.load(sys.stdin)]"

# 檢查是否有 agent 超過 2 小時未回應
curl -s http://127.0.0.1:3100/api/companies/{companyId}/agents | \
  python3 -c "
import sys,json
from datetime import datetime, timezone, timedelta
for a in json.load(sys.stdin):
    hb = a.get('lastHeartbeatAt')
    if hb:
        last = datetime.fromisoformat(hb.replace('Z','+00:00'))
        if datetime.now(timezone.utc) - last > timedelta(hours=2):
            print(f'WARNING: {a[\"name\"]} 超過 2 小時未回應')
"
```

### 1.4 錯誤日誌收集

Workers 中統一錯誤格式：

```typescript
// middleware/error-handler.ts
export async function errorHandler(c: Context, next: Next) {
  try {
    await next()
  } catch (e) {
    console.error(JSON.stringify({
      level: 'error',
      endpoint: c.req.path,
      method: c.req.method,
      error: e.message,
      stack: e.stack,
      timestamp: new Date().toISOString()
    }))
    return c.json({ error: 'Internal Server Error' }, 500)
  }
}
```

使用 `wrangler tail` 即時查看：
```bash
wrangler tail --format=json | jq 'select(.level == "error")'
```

## 二、Agent 故障恢復協議

### 2.1 故障檢測

| 故障類型 | 檢測方式 | 自動恢復 |
|---|---|---|
| Heartbeat 超時 | 超過 2 小時無 heartbeat | 自動重觸發 |
| Agent 狀態卡在 running | 超過 1 小時仍為 running | 通知董事會 |
| CLI 連線失敗 | `claude -p` 返回非零 | 重試 3 次後暫停 |
| API 限流 | 429 錯誤 | 自動 backoff |

### 2.2 恢復步驟

**場景 1：Agent heartbeat 超時**
```bash
# 1. 檢查 agent 狀態
curl -s http://127.0.0.1:3100/api/agents/{agentId}

# 2. 手動觸發 heartbeat
curl -X POST http://127.0.0.1:3100/api/agents/{agentId}/heartbeat/invoke

# 3. 如果仍然失敗，檢查 CLI
NODE_TLS_REJECT_UNAUTHORIZED=0 claude -p "hello"

# 4. 如果 CLI 也失敗，檢查網路/代理
curl -s https://api.anthropic.com/v1/messages -H "x-api-key: test" 2>&1
```

**場景 2：CEO 故障（單點故障緩解）**
```
1. CTO 小李自動接管調度職責（backup orchestrator）
2. PM 和 QA 可以直接溝通，不等 CEO 批准 Sprint Contract
3. 超過 4 小時無 CEO 回應 → 升級至董事會（人類）
4. 董事會手動建立任務分配給其他 agent
```

**場景 3：Paperclip server 崩潰**
```bash
# 1. 重啟
cd paperclip && pnpm dev

# 2. 檢查健康狀態
curl http://127.0.0.1:3100/api/health

# 3. 嵌入式 PostgreSQL 資料在 ~/.paperclip/instances/default/db/
# 自動恢復，不需要手動介入
```

### 2.3 CEO 單點故障緩解

> 審查報告指出 CEO 是災難性單點故障。

**緩解措施：**
1. CTO 小李 的 instructions 中加入 backup orchestrator 職責
2. Sprint Contract 不需要 CEO 批准即可開工（超過 24 小時 CEO 無回應時）
3. QA 評分報告可以直接抄送董事會
4. 建立「緊急通道」— 任何 agent 可以建立 priority=critical 的任務直達董事會

## 三、部署回滾 Runbook

### 3.1 Workers API 回滾

```bash
# 查看部署歷史
wrangler deployments list

# 回滾到上一版本
wrangler rollback

# 回滾到指定版本
wrangler rollback --version-id={versionId}
```

### 3.2 Cloudflare Pages 回滾

```bash
# 查看部署歷史
wrangler pages deployment list --project-name={projectName}

# Pages 無法直接 rollback，需要重新部署舊版本
git checkout {舊 commit} -- apps/web/
cd apps/web && pnpm build && wrangler pages deploy dist --project-name={projectName}
```

### 3.3 D1 資料庫回滾

```bash
# D1 不支援原生 rollback，使用備份
# Paperclip 每 60 分鐘自動備份到 ~/.paperclip/instances/default/data/backups/

# 手動備份
wrangler d1 export {dbName} --remote --output=backup-$(date +%Y%m%d).sql

# 恢復（需要先刪除再重建）
wrangler d1 execute {dbName} --remote --file=backup.sql
```

### 3.4 支付異常處理

```
1. ECPay Webhook 收到付款通知但會員未開通：
   → 查詢 D1 orders 表找到對應訂單
   → 手動更新 subscriptions 表
   → 通知用戶

2. 用戶重複付款：
   → ECPay 後台查詢交易記錄
   → 確認重複後聯繫 ECPay 退款
   → 記錄事件到 payment_incidents 表

3. Webhook 端點掛了：
   → ECPay 會重試 3 次（間隔 1/5/15 分鐘）
   → 修復端點後手動對帳
```

### 3.5 緊急聯繫

| 系統 | 聯繫方式 |
|---|---|
| Cloudflare 問題 | https://dash.cloudflare.com → Support |
| ECPay 技術問題 | sysanalydep.sa@ecpay.com.tw |
| Paperclip 問題 | https://github.com/paperclipai/paperclip/issues |
