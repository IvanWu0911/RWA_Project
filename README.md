# 房產鏈金術 — RWA 不動產代幣化交易平台

> Real World Asset (RWA) tokenization platform：將台灣不動產租金收益權代幣化，
> 提供 KYC 審核、鏈上發行、限價／市價撮合、租金分潤與鏈上／鏈下自動對帳的完整流程。

**大學資管系專題（4 人團隊）** ｜ 整理與維護：[IvanWu0911](https://github.com/IvanWu0911)

> **我在本專題負責的部分**：智能合約與區塊鏈相關後端
> — ERC-3643 合規代幣合約撰寫與部署、ethers.js 鏈上互動、Nonce 管理與失敗後的
> 自動回復、節點失憶偵測與狀態重建排程，以及鏈上／鏈下對帳 `reconcile()` 與其故障注入測試。
> 對應目錄：[`blockchain/`](blockchain/) 與 [`rwa-backend/src/blockchain/`](rwa-backend/src/blockchain/)。

![建案詳情：K 線、掛單簿與下單面板](docs/screenshots/investor-property-detail.png)

---

## 專案架構

```
.
├── blockchain/              # Hardhat + Solidity（ERC-3643 合規代幣）
├── rwa-backend/             # NestJS 後端 API（撮合、KYC、分潤、鏈上對帳）
│   └── src/blockchain/      #   合約部署、轉帳、Nonce 管理、reconcile()
├── frontend/                # React + Vite 前端（投資人／業務端／技術端）
├── crawler/                 # Python 爬蟲：抓取台灣實價登錄與租屋行情
├── test-logs/               # 自動化測試報告與原始 log（89 筆測試）
├── docs/                    # 專題文件書（PDF / DOCX，95 頁）與系統截圖
├── docker-compose.yml       # 本機 PostgreSQL
└── init-db.sql
```

## 技術棧

| 層級 | 技術 |
|---|---|
| 區塊鏈 | Solidity、Hardhat、ethers.js v6、ERC-3643 合規代幣標準 |
| 後端 | NestJS 11、TypeORM、Passport-JWT、Throttler、Schedule |
| 前端 | React 18、Vite、TypeScript、Radix UI / shadcn、MUI、lightweight-charts（K 線） |
| 資料庫 | PostgreSQL 15（本機 Docker）／ Supabase（雲端） |
| 爬蟲 | Python、Playwright、psycopg2 |
| 測試 | Jest（單元／整合／故障注入）、Hardhat test |

## 主要功能

- **合規代幣（ERC-3643）**：身分註冊（OnchainID）與轉帳限制，僅通過 KYC 的錢包可持有代幣
- **鏈上／鏈下對帳**：`reconcile()` 以 `queryFilter` 讀取 `Transfer` 事件全量重建鏈上真實持倉，
  比對資料庫並偵測未追蹤轉帳；具自動修復與人工確認兩種模式
- **節點失憶自動重建**：每 30 秒排程檢查合約 code 是否仍存在（雲端節點休眠會清空鏈上狀態），
  以 `isRecovering` 旗標避免重複部署，並區分「RPC 瞬斷」與「合約真的消失」
- **Nonce 脫節回復**：鏈上呼叫失敗時僅在 `NONCE_EXPIRED` / `REPLACEMENT_UNDERPRICED`
  才 reset NonceManager，避免有交易在途時造成重號
- **撮合引擎**：限價／市價單、滑價計算、台灣 K 線紅漲綠跌慣例
- **租金分潤**：依 `payout_cycle_days` 週期分配，含重複發放鎖定與合規週期警示
- **三種角色**：投資人（下單、持倉、CSV 稽核匯出）、業務端（KYC 審核、租金監管）、
  技術端（合約部署、Oracle 監控、系統健康度、鏈上對帳）

## 系統畫面

### 投資人端

| 登入頁（三種示範角色） | 持倉總覽 |
|---|---|
| ![登入頁](docs/screenshots/login.png) | ![持倉總覽](docs/screenshots/investor-portfolio.png) |

| 房產市場（縣市篩選、即時現價） | 交易紀錄（掛單 / 歷史 / CSV 稽核匯出） |
|---|---|
| ![房產市場](docs/screenshots/investor-market.png) | ![交易紀錄](docs/screenshots/investor-transactions.png) |

### 業務端 — 租金監管與收益發放

![業務端儀表板](docs/screenshots/banker-dashboard.png)

### 技術端 — 系統監控與合約控制

![技術端儀表板](docs/screenshots/technician-dashboard.png)

## 專題文件

完整的系統文件書（摘要、系統設計、ER 模型、循序圖、API 授權矩陣、負載測試、SWOT 分析等，共 95 頁）：

- 📄 [房產鏈金術 RWA房產代幣化交易系統文件書.pdf](docs/房產鏈金術%20RWA房產代幣化交易系統文件書.pdf) — 可直接在 GitHub 上預覽
- 📝 [Word 原檔（.docx）](docs/房產鏈金術%20RWA房產代幣化交易系統文件書.docx)

## 快速開始

### 1. 資料庫

```bash
docker compose up -d
```

### 2. 區塊鏈節點與合約

```bash
cd blockchain
npm install
npx hardhat compile
npx hardhat node          # 另開終端機保持執行
```

### 3. 後端

```bash
cd rwa-backend
pnpm install
# 建立 .env，至少需要：DATABASE_URL、JWT_SECRET、FRONTEND_URL、RPC_URL、ADMIN_KEY
pnpm run start:dev        # http://localhost:3001
```

### 4. 前端

```bash
cd frontend
npm install
npm run dev               # http://localhost:5173
```

示範帳號：`technician`（技術端）、`banker`（業務端）、`investor`（投資人）。

### 5. 爬蟲（選用）

```bash
cd crawler
pip install -r requirements.txt
playwright install
# 建立 .env，設定 DATABASE_URL
python main.py
```

## 測試

```bash
cd rwa-backend
pnpm run test             # 單元／負向／併發／故障注入
pnpm run test:e2e         # 整合／權限

cd ../blockchain
npm test                  # 智能合約（ERC-3643）
```

後端 54 筆 + e2e 15 筆 + 合約 20 筆，共 **89 筆**。
完整測試計畫、結果與原始 log 見 [`test-logs/`](test-logs/)。

其中鏈上／鏈下狀態一致性的五個故障情境（服務重啟、重複事件、Nonce 衝突、區塊重組、
通知失敗）另有獨立報告：[`test-logs/故障注入測試_五情境.md`](test-logs/)。

## 環境變數

本專案**不會**把 `.env` 提交進版本庫。後端啟動時若缺少 `JWT_SECRET` 會直接中止；
缺少 `ADMIN_KEY` 時會拒絕以公開的 Hardhat 測試私鑰啟動鏈上功能；爬蟲缺少 `DATABASE_URL` 亦同。
請依上方「快速開始」自行建立。

## 授權

Private — All rights reserved.
