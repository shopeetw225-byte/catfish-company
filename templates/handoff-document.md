# Sprint 交接文件模板
> Template Version: v1.0 | Last Updated: 2026-03-29

> Generator 在 sprint 結束後提交給 Evaluator

## Sprint 名稱
[產品名 — Sprint N]

## 實作了什麼
1. [功能 1] — [簡述]
2. [功能 2] — [簡述]
3. [功能 3] — [簡述]

## 修改了哪些文件
```
apps/web/src/pages/XXX.tsx     — 新增頁面
apps/web/src/components/XXX.tsx — 新增組件
apps/api/src/routes/XXX.ts     — 新增 API
db/migrations/XXX.sql          — 新增資料表
```

## 如何本地測試

```bash
# 1. 安裝依賴
pnpm install

# 2. 啟動後端
cd apps/api && wrangler dev

# 3. 啟動前端
cd apps/web && pnpm dev

# 4. 打開瀏覽器
open http://localhost:5173
```

## 測試帳號
- Email: test@example.com
- Password: test1234
- ECPay 測試信用卡號: 4311-9522-2222-2222

## 已知問題
1. [問題 1] — [影響程度]
2. [問題 2] — [影響程度]

## 部署方式

```bash
# 前端
cd apps/web && pnpm build && wrangler pages deploy dist --project-name=XXX

# 後端
cd apps/api && wrangler deploy

# D1 Migration
wrangler d1 execute XXX --remote --file=db/migrations/XXX.sql
```

## 備註
[任何 Evaluator 需要知道的額外資訊]
