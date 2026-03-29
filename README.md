# 貓魚印象有限公司 — AI Agent 軟件公司運作藍圖

> 一家由 AI Agent 團隊運營的軟件公司，目標：**賺大錢**。

## 公司簡介

貓魚印象有限公司是一家以 AI Agent 為核心勞動力的軟件公司，採用 [Anthropic Harness Design](https://www.anthropic.com/engineering/harness-design-long-running-apps) 的 GAN 對抗式架構，嚴格分離規劃、實作、驗證三個環節，確保產出品質。

## 核心理念

1. **賺錢優先** — 每個功能都要問「這能讓用戶付錢嗎？」
2. **GAN 對抗架構** — 做事的人不能評判自己的工作
3. **Sprint Contract 先行** — 開工前先定義「完成長什麼樣」
4. **Context Reset** — 每個 sprint 結束後清空上下文，用結構化文件傳遞
5. **可量化評分** — 不是模糊的「通過/不通過」，是具體分數

## 組織架構

```
董事會（人類）
  └── CEO 小張（Orchestrator）
        ├── PM 小陳（Planner）
        │     → 把方向擴展成完整 PRD
        │     → 與 Evaluator 協商 Sprint Contract
        │
        ├── CTO 小李（Generator Lead + Backup Orchestrator + DevOps）
        │     ├── 工程師 小林（Generator — 全端）
        │     └── 支付專家 小錢（Generator — ECPay）
        │     → 按 Sprint Contract 實作
        │     → 不能 review 自己的代碼
        │     → CEO 故障時自動接管調度（4 小時超時）
        │
        └── QA 小趙（Evaluator）
              → 獨立驗證，偏向懷疑
              → 5 維度量化評分（每季校準一次）
              → 直接向 CEO 匯報（獨立於 CTO）
```

## 技術棧

| 層級 | 技術 |
|---|---|
| 前端 | React 19 + Vite 7 + Tailwind CSS v4 |
| 後端 | Cloudflare Workers + Hono |
| 資料庫 | Cloudflare D1 (SQLite) |
| Session | Cloudflare KV |
| 支付 | 綠界支付 ECPay |
| AI 運算 | Cloudflare Workers AI |
| Agent 平台 | [Paperclip](https://github.com/paperclipai/paperclip) |

## 目標市場

- **台灣**（唯一市場）— 繁體中文，綠界支付

## 目錄結構

```
catfish-company/
├── README.md                              # 本文件
├── docs/
│   ├── harness-architecture.md            # Harness Design 架構詳解
│   ├── sprint-workflow.md                 # Sprint 流程規範
│   ├── evaluation-criteria.md             # 評分標準
│   ├── tech-stack.md                      # 技術棧決策
│   ├── business-model.md                  # 商業模式與收入評估
│   ├── paperclip-setup.md                 # Paperclip 部署指南
│   ├── operations.md                      # 監控、故障恢復、部署回滾
│   ├── agent-communication-protocol.md    # Agent 間通訊協議與狀態機
│   └── review-response.md                # 架構審查回應記錄
├── agents/
│   ├── ceo/AGENTS.md            # CEO 小張 — Orchestrator
│   ├── pm/AGENTS.md             # PM 小陳 — Planner
│   ├── cto/AGENTS.md            # CTO 小李 — Generator Lead
│   ├── engineer/AGENTS.md       # 工程師 小林 — Generator
│   ├── payment/AGENTS.md        # 支付專家 小錢 — Generator (Payment)
│   └── qa/AGENTS.md             # QA 小趙 — Evaluator
└── templates/
    ├── prd-template.md          # PRD 模板 (v1.0)
    ├── sprint-contract.md       # Sprint Contract 模板 (v1.0)
    ├── handoff-document.md      # 交接文件模板 (v1.0)
    └── evaluation-report.md     # 評分報告模板 (v1.0)
```

### 文件關係圖

```
sprint-workflow.md ──引用──→ agent-communication-protocol.md（狀態機）
       │                            │
       │                            ├──→ templates/sprint-contract.md
       │                            ├──→ templates/handoff-document.md
       │                            └──→ templates/evaluation-report.md
       │
       └──引用──→ evaluation-criteria.md（評分標準）

operations.md ──引用──→ agents/cto/AGENTS.md（DevOps 職責）
       │
       └──引用──→ paperclip-setup.md（安全配置）

business-model.md ──引用──→ evaluation-criteria.md（產品提案評估）
```

## 快速開始

1. 安裝 [Paperclip](https://github.com/paperclipai/paperclip)
2. **閱讀安全注意事項**：`docs/paperclip-setup.md` 的安全章節
3. 建立公司，匯入 `agents/` 下的 AGENTS.md 作為各 agent 的 instructions
4. 閱讀 `docs/agent-communication-protocol.md` 了解 Agent 間如何協作
5. 建立第一個產品任務，觸發 CEO heartbeat
6. 觀察 Sprint 流程自動運轉

## 審查記錄

| 日期 | 版本 | 評分 | 狀態 |
|---|---|---|---|
| 2026-03-29 | v1 | 2.2/5.0 | 未通過 |
| 2026-03-29 | v2 | 3.3/5.0 | 未通過（差 0.2） |
| 2026-03-29 | v3 | 待審 | 修復二審全部問題 |

詳見 `docs/review-response.md`

## 授權

MIT License — 貓魚印象有限公司 2026
