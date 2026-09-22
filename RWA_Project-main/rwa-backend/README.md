# rwa-backend

房產鏈金術 RWA 平台的 NestJS 後端。負責使用者／KYC、物件管理、撮合與結算、租金分潤、
鏈上互動（ethers.js → Hardhat / ERC-3643）以及鏈上／鏈下對帳。

**作者：[IvanWu0911](https://github.com/IvanWu0911)**

## 模組

| 目錄 | 職責 |
|---|---|
| `auth/` | JWT 登入、角色守衛（investor / banker / technician） |
| `users/` | 使用者與 KYC 狀態 |
| `properties/` | 不動產物件、租金週期（`payout_cycle_days`） |
| `transactions/` | 限價／市價撮合、滑價、結算、CSV 稽核匯出 |
| `portfolio/` | 持倉與損益 |
| `blockchain/` | 合約部署、轉帳、Nonce 管理、節點自動重建、`reconcile()` 對帳 |
| `notifications/` | 使用者通知（commit 後獨立寫入，不回滾交易） |
| `system/` | 系統日誌、健康度、爬蟲回報 |
| `seed/` | 示範資料 |
| `entities/` | TypeORM 實體 |

## 環境變數

啟動前請在本目錄建立 `.env`（**不要提交**）：

```
DATABASE_URL=postgresql://user:password@localhost:5433/rwa_db
JWT_SECRET=<隨機字串>
FRONTEND_URL=http://localhost:5173
```

缺少 `JWT_SECRET` 時服務會直接中止。

## 指令

```bash
pnpm install

pnpm run start:dev     # 開發模式（watch）
pnpm run start:prod    # 執行 dist/main
pnpm run build         # 先編譯 ../blockchain 合約，再 nest build

pnpm run test          # 單元／負向／併發／故障注入
pnpm run test:e2e      # 整合／權限
pnpm run test:cov      # 覆蓋率
pnpm run lint
```

測試報告與原始 log 見 [`../test-logs/`](../test-logs/)。
