# 技術棧決策

## 核心原則：全部在 Cloudflare 生態系統內

不引入 AWS/GCP/Vercel/Netlify。原因：
- 成本低（大量免費額度）
- 全球 CDN 延遲低
- 統一平台降低運維複雜度
- Edge Computing 離用戶更近

## 前端

| 技術 | 版本 | 選擇原因 |
|---|---|---|
| React | 19 | 生態成熟，招人容易 |
| Vite | 7 | 開發體驗好，構建快 |
| Tailwind CSS | v4 | Utility-first，不寫自定義 CSS |

### 前端規範
- 使用 TypeScript，不用 JavaScript
- 組件用函數式 + Hooks
- 狀態管理用 React Context 或 Zustand（不用 Redux）
- i18n 用 react-i18next

## 後端

| 技術 | 用途 |
|---|---|
| Cloudflare Workers | 無伺服器執行環境 |
| Hono | 輕量 Web 框架（專為 Edge 設計） |
| Cloudflare D1 | SQLite 資料庫 |
| Cloudflare KV | Key-Value 儲存（Session、快取） |
| Workers AI | AI 推理（Llama、Mistral 等） |

### 後端規範
- 使用 TypeScript
- RESTful API 設計
- 所有端點都要驗證 JWT token
- Rate limiting 用 KV 實作
- 錯誤回傳統一格式：`{ error: string, code: number }`

## 支付

| 技術 | 用途 |
|---|---|
| 綠界支付 ECPay | 台灣市場（信用卡/ATM/超商） |
| Stripe | 國際市場（M3 階段） |

### 支付安全規範
- HashKey/HashIV 只存在 Workers Secrets，不進 git
- CheckMacValue 用 timing-safe comparison
- Webhook 回調驗證簽名後才處理
- 訂單用 MerchantTradeNo 做冪等

## 部署

```bash
# 前端部署到 Cloudflare Pages
cd apps/web && pnpm build && wrangler pages deploy dist --project-name=產品名

# 後端部署到 Workers
cd apps/api && wrangler deploy

# D1 Migration
wrangler d1 execute 資料庫名 --remote --file=db/migrations/001_init.sql
```

## 本地開發

```bash
# 啟動後端（Workers 模擬）
cd apps/api && wrangler dev

# 啟動前端（Vite dev server）
cd apps/web && pnpm dev

# 前端 proxy 到後端
# vite.config.ts 中設定 /api → localhost:8787
```

## 專案結構模板

```
project-name/
├── apps/
│   ├── web/                  # 前端（React + Vite）
│   │   ├── src/
│   │   │   ├── components/   # UI 組件
│   │   │   ├── pages/        # 頁面
│   │   │   ├── hooks/        # Custom hooks
│   │   │   ├── lib/          # 工具函數
│   │   │   ├── i18n/         # 多語系
│   │   │   └── main.tsx
│   │   ├── index.html
│   │   ├── vite.config.ts
│   │   └── package.json
│   └── api/                  # 後端（Workers + Hono）
│       ├── src/
│       │   ├── routes/       # API 路由
│       │   ├── middleware/   # 中間件（auth、cors、rate-limit）
│       │   ├── db/           # D1 查詢
│       │   └── index.ts
│       ├── wrangler.toml
│       └── package.json
├── db/
│   └── migrations/           # D1 migration SQL
├── pnpm-workspace.yaml
└── package.json
```
