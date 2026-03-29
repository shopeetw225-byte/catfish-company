# Paperclip 部署指南

## 什麼是 Paperclip

[Paperclip](https://github.com/paperclipai/paperclip) 是 AI Agent 公司的控制平面，管理 agent 團隊、任務、審批流程。

## 安裝

```bash
# Clone
git clone https://github.com/paperclipai/paperclip.git
cd paperclip

# 安裝依賴
pnpm install

# 修復 Node 22 的 tsx 問題
# 在 server/scripts/dev-watch.ts 中
# 把 require.resolve("tsx/dist/cli.mjs") 改為 require.resolve("tsx/cli")

# 啟動開發模式
pnpm dev
# UI: http://127.0.0.1:3100
```

## 建立公司和 Agent

### 方式一：UI 操作
1. 打開 http://127.0.0.1:3100
2. 建立公司「貓魚印象有限公司」
3. 逐個建立 Agent

### 方式二：API 操作（推薦）

```bash
# 建立 Agent（以 CEO 為例）
curl -X POST "http://127.0.0.1:3100/api/companies/{companyId}/agents" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "CEO 小張",
    "role": "ceo",
    "adapterType": "claude_local",
    "adapterConfig": {
      "model": "claude-sonnet-4-6",
      "maxTurnsPerRun": 300,
      "dangerouslySkipPermissions": true,
      "instructionsBundleMode": "managed",
      "env": {"NODE_TLS_REJECT_UNAUTHORIZED": "0"}
    }
  }'
```

### Agent Instructions

每個 agent 的 instructions 文件位置：
```
~/.paperclip/instances/default/companies/{companyId}/agents/{agentId}/instructions/AGENTS.md
```

把 `agents/` 目錄下對應角色的 AGENTS.md 複製到上述路徑。

## 組織架構

```
CEO（ceo）→ 不設 reportsTo
  ├── PM（pm）→ reportsTo: CEO
  ├── CTO（cto）→ reportsTo: CEO
  │     ├── 工程師（engineer）→ reportsTo: CTO
  │     └── 支付專家（engineer）→ reportsTo: CTO
  └── QA（qa）→ reportsTo: CEO  ← 重要：QA 直接向 CEO 匯報，獨立於 CTO
```

## 關鍵配置

### Heartbeat 間隔
```bash
# CEO 每 5 分鐘醒一次
curl -X PATCH "http://127.0.0.1:3100/api/agents/{agentId}" \
  -H "Content-Type: application/json" \
  -d '{"runtimeConfig":{"heartbeat":{"enabled":true,"intervalSec":300}}}'
```

### 環境變數（中國大陸用戶）
如果使用代理/VPN，需要在 adapter 配置中加入：
```json
"env": {"NODE_TLS_REJECT_UNAUTHORIZED": "0"}
```

### 觸發 Heartbeat
```bash
curl -X POST "http://127.0.0.1:3100/api/agents/{agentId}/heartbeat/invoke"
```

### 建立任務
```bash
curl -X POST "http://127.0.0.1:3100/api/companies/{companyId}/issues" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "任務標題",
    "description": "詳細描述",
    "status": "todo",
    "priority": "critical",
    "assigneeAgentId": "{agentId}"
  }'
```

## 可用角色

Paperclip 只接受以下角色值：
`ceo` `cto` `cmo` `cfo` `engineer` `designer` `pm` `qa` `devops` `researcher` `general`

## 注意事項

1. Agent 需要能訪問 Anthropic API — `claude_local` adapter 會呼叫 `claude -p`
2. 中國大陸需要代理，但要確保 `NODE_TLS_REJECT_UNAUTHORIZED=0`
3. Codex adapter 在中文路徑下有 UTF-8 header bug，確保工作目錄是英文路徑
