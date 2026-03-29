# Agent 間通訊協議

> 審查報告指出：「Agent 間通訊協議完全隱式」。本文件定義明確的協議。

## 通訊介質：Paperclip Issue System

所有 Agent 間通訊通過 Paperclip 的 Issue（任務）系統進行。不使用隱式文件交換。

### 狀態機

```
[backlog] → [todo] → [in_progress] → [in_review] → [done]
                ↑         |                |
                |         ↓                ↓
                +--- [blocked]        [in_progress]（打回重做）
                                           |
                                           ↓
                                     [cancelled]
```

### 有效狀態轉換

| 從 | 到 | 誰可以轉 | 條件 |
|---|---|---|---|
| backlog | todo | CEO, PM | CEO 批准產品方向 |
| todo | in_progress | 被分配的 agent | 開始工作 |
| in_progress | blocked | 任何 agent | 遇到外部阻礙，必須留言說明 |
| blocked | in_progress | 任何 agent | 阻礙解除 |
| in_progress | in_review | Generator | 實作完成，附交接文件 |
| in_review | done | Evaluator（QA） | 評分 ≥ 3.5 |
| in_review | in_progress | Evaluator（QA） | 評分 < 3.5，附評分報告 |
| 任何 | cancelled | CEO, 董事會 | 取消任務 |

### 無效轉換（禁止）

- `todo → done`（不能跳過實作和 review）
- `in_progress → done`（不能跳過 review）
- Generator 自己把 `in_review → done`（Evaluator 獨立性）

## 任務分配協議

### CEO → 下屬
```json
{
  "title": "任務標題（繁體中文）",
  "description": "詳細描述，包含背景和期望產出",
  "assigneeAgentId": "目標 agent ID",
  "priority": "critical | high | medium | low",
  "parentId": "父任務 ID（如果是子任務）"
}
```

### Generator → Evaluator（in_review 轉換時）
必須在 issue comment 中附上交接文件（使用 `templates/handoff-document.md` 格式）。

### Evaluator → CEO（評分完成時）
必須在 issue comment 中附上評分報告（使用 `templates/evaluation-report.md` 格式）。

## Sprint Contract 協議

### 協商流程
```
1. PM 建立 issue: "Sprint Contract: [產品名 Sprint N]"
   - 附 PRD 內容
   - assignee: QA

2. QA 在 comment 中回覆評分標準
   - 每項驗收條件 + 1-5 分定義

3. PM 在 comment 中確認或提出修改

4. 雙方達成共識後，QA 留言 "Sprint Contract 已確認"

5. CEO 留言 "Sprint Contract 已批准" → 狀態改為 done
   （如果 CEO 超過 24 小時無回應，CTO 可代為批准）

6. CTO 根據已批准的 Sprint Contract 建立實作子任務
```

### Sprint Contract 存儲位置
以 issue document 形式存儲在對應的 Sprint issue 中。

## 緊急通道

任何 agent 遇到以下情況可以直接建立 `priority: critical` 的任務，自動通知董事會：

1. 安全漏洞
2. 支付系統故障
3. 生產環境掛了
4. Agent 間死鎖（互相等待）
5. CEO 超過 4 小時無回應

## 共享文件約定

| 文件類型 | 存儲方式 | 誰負責 |
|---|---|---|
| PRD | issue document (key: "prd") | PM |
| Sprint Contract | issue document (key: "contract") | PM + QA |
| 交接文件 | issue comment | CTO / Generator |
| 評分報告 | issue comment | QA |
| 技術架構決策 | issue document (key: "tech-design") | CTO |
| 部署記錄 | issue comment | DevOps / CTO |
