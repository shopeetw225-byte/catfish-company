# Harness Design 架構詳解

> 參考：[Anthropic - Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps)

## 核心問題

AI Agent 長時間運行面臨兩個問題：

1. **Context 退化** — 對話歷史累積後，agent 容易草草收工
2. **自我評價偏差** — agent 傾向高估自己的工作品質

## 解法：GAN 對抗式架構

借鑑生成對抗網路（GAN）的思路，將「生成」和「評判」分離：

- **Generator**（生成者）— 負責實作，專注於產出
- **Evaluator**（評判者）— 獨立驗證，偏向懷疑

> "Separating the agent doing the work from the agent judging it enables meaningful feedback loops."

### 為什麼不能自我評價？

因為 Agent 會「confidently praise the work — even when, to a human observer, the quality is obviously mediocre」。單獨的 Evaluator 可以被調教得更嚴格，而讓 Generator 自我批判效果很差。

## 三角色系統

### 1. Planner（規劃者）→ PM 小陳

- 把 1-4 句的簡短需求擴展成完整產品規格
- 主動融入 AI 功能機會
- 防止規格錯誤傳播到實作階段
- **不過度指定實作細節**，留給 Generator 技術決策權

### 2. Generator（生成者）→ CTO 小李 + 工程師 小林 + 支付專家 小錢

- 按 Sprint Contract 逐功能實作
- 使用 git 做版本控制
- 在提交 QA 前做基本自檢（但不算正式 review）
- **不能評判自己的最終品質**

### 3. Evaluator（評判者）→ QA 小趙

- 用自動化工具（Playwright）像真實用戶一樣測試
- 測試 UI 功能、API 端點、資料庫狀態
- 按硬性門檻評分
- **需要大量調教避免評分過於寬鬆**

## Sprint Contract 機制

開工前，Generator 和 Evaluator 先協商：

1. 這個 sprint 要交付什麼
2. 怎麼驗證算通過
3. 可測試的成功標準

這樣做的好處：
- 避免「做完了但不是我要的」
- 讓 Evaluator 有明確的評分依據
- Generator 知道什麼算完成，不會過度工程

## Context Reset vs Compaction

| 方法 | 優點 | 缺點 |
|---|---|---|
| **Compaction**（壓縮摘要） | 保持連續性 | Context anxiety 持續累積，agent 草草收工 |
| **Context Reset**（完全重置） | 乾淨的上下文，專注當前任務 | 需要結構化交接文件，增加延遲 |

**結論**：較舊的模型需要 Context Reset；Claude Opus 4.6 基本消除了 context anxiety，可以用單一連續 session。

## Harness 演化原則

> "Every component in a harness encodes an assumption about what the model can't do on its own."

**當前決策：維持完整 Harness 架構，不做簡化。** 公司尚未出貨任何產品，現階段討論簡化架構是過早優化。等第一個產品成功上線並運行穩定後，再評估哪些組件可以移除。

## 成本參考

| 項目 | Solo Agent | Full Harness |
|---|---|---|
| 復古遊戲 | 20 min / $9 | 6 hr / $200 |
| DAW 音頻工作站 | — | 3h50m / $124.70 |

Harness 更貴但品質差距巨大。Solo agent 產出功能殘缺，Harness 產出功能完整可用。
