---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: "00000000000000000000000000000000"
    PropagateID: "00000000000000000000000000000000"
    ReservedCode1: 304602210085b865b159ca4ad2ff0161c4f1c21e1b0e981d51b41bbc15c51b9952295d734c022100f5364f6a8dd60b4bab12c5ada75a8cab31e9b2dfdf282a99bce8b8931288c771
    ReservedCode2: 3045022078b5463e4ee8c8f6d1e57b85171c8e7e9e4ed311e503ae97c408d6e53adebc65022100a86d15240943f34f1c161702b01a6423b1bd896776e2fd3b06c2fc98619bddeb
---

# 股票年报数据自动获取和更新服务

本服务提供股票年报数据的自动获取、存储和更新功能，支持从同花顺网站抓取上市公司财务数据，并提供RESTful API接口。

## 功能特性

- **自动数据获取**：从同花顺获取股票基础信息和历史财务数据
- **智能缓存机制**：24小时自动缓存，避免重复请求
- **定时更新**：支持cron表达式配置自动更新任务
- **观察列表**：管理关注的股票列表
- **RESTful API**：完整的API接口支持
- **日志记录**：详细的运行日志便于监控和调试

## 目录结构

```
server/
├── src/
│   ├── index.ts              # 服务入口文件
│   ├── routes/
│   │   └── stocks.ts        # 股票数据API路由
│   ├── services/
│   │   ├── thsScraper.ts    # 同花顺数据爬取服务
│   │   ├── scheduler.ts     # 自动更新调度服务
│   │   └── dataStore.ts     # 数据存储服务
│   ├── utils/
│   │   └── logger.ts        # 日志工具
│   ├── scripts/
│   │   ├── fetchStockData.ts  # 批量获取数据脚本
│   │   └── updateAllData.ts   # 更新所有数据脚本
│   └── types/
│       └── stock.ts         # TypeScript类型定义
├── data/                     # 数据存储目录（自动创建）
├── logs/                     # 日志目录（自动创建）
├── package.json
├── tsconfig.json
├── .env                      # 环境变量配置
└── README.md
```

## 快速开始

### 1. 安装依赖

```bash
cd server
npm install
```

### 2. 配置环境变量

编辑 `.env` 文件：

```env
# 服务器端口
PORT=3001

# 同花顺基础URL
THS_BASE_URL=https://basic.10jqka.com.cn

# 请求间隔（毫秒）
THS_API_DELAY=2000

# 请求超时（毫秒）
THS_REQUEST_TIMEOUT=30000

# 自动更新调度配置
# 调度表达式（默认每天凌晨2点执行）
SCHEDULE_CRON=0 2 * * *

# 是否启用自动更新（true/false）
AUTO_UPDATE_ENABLED=false

# 日志级别
LOG_LEVEL=info

# 数据目录
DATA_DIR=data
```

### 3. 启动服务

开发模式（使用ts-node）：

```bash
npm run dev
```

生产模式（需要先编译）：

```bash
npm run build
npm start
```

## API接口文档

### 基础信息

- **Base URL**: `http://localhost:3001`
- **响应格式**: JSON
- **时间戳**: 所有响应包含ISO格式时间戳

### 股票数据接口

#### 获取股票数据

```http
GET /api/stocks/:symbol
```

**参数**：
- `symbol` (路径参数): 股票代码，如 `600519`
- `cache` (查询参数): 是否使用缓存，默认 `true`，设置为 `false` 强制刷新
- `fetch` (查询参数): 是否从同花顺获取新数据，默认 `false`

**示例**：
```bash
curl http://localhost:3001/api/stocks/600519
curl http://localhost:3001/api/stocks/600519?fetch=true
```

**响应**：
```json
{
  "success": true,
  "data": {
    "symbol": "600519",
    "name": "贵州茅台",
    "industry": "酿酒食品",
    "area": "贵州",
    "listDate": "2001-08-27",
    "market": "主板",
    "exchange": "上海证券交易所",
    "currency": "CNY",
    "historicalData": [
      {
        "year": 2023,
        "metrics": {
          "totalAssets": 2544552800000,
          "totalLiabilities": 542192000000,
          "currencyFunds": 1723058000000,
          "revenue": 1472266300000,
          "netProfit": 747537730000,
          "roe": 35.53,
          "debtAssetRatio": 21.3
        },
        "reportType": "年报"
      }
    ],
    "lastUpdated": "2024-01-15T10:30:00.000Z",
    "dataSource": "同花顺"
  },
  "timestamp": "2024-01-15T10:30:00.000Z"
}
```

#### 搜索股票

```http
GET /api/stocks/search/:keyword
```

**示例**：
```bash
curl http://localhost:3001/api/stocks/search/贵州茅台
```

#### 获取所有股票列表

```http
GET /api/stocks/list/all
```

#### 获取观察列表

```http
GET /api/stocks/list/watchlist
```

#### 从同花顺获取股票数据

```http
POST /api/stocks/fetch
Content-Type: application/json

{
  "symbols": ["600519", "000001", "300750"]
}
```

**示例**：
```bash
curl -X POST http://localhost:3001/api/stocks/fetch \
  -H "Content-Type: application/json" \
  -d '{"symbols": ["600519", "000001"]}'
```

#### 手动更新单只股票

```http
POST /api/stocks/update/:symbol
```

**示例**：
```bash
curl -X POST http://localhost:3001/api/stocks/update/600519
```

#### 添加股票到观察列表

```http
POST /api/stocks/watchlist
Content-Type: application/json

{
  "symbols": ["600519", "000001"]
}
```

#### 从观察列表移除

```http
DELETE /api/stocks/watchlist
Content-Type: application/json

{
  "symbols": ["600519"]
}
```

#### 删除股票数据

```http
DELETE /api/stocks/:symbol
```

#### 获取存储统计

```http
GET /api/stocks/stats/all
```

### 调度服务接口

#### 获取调度状态

```http
GET /api/scheduler/status
```

#### 手动触发更新任务

```http
POST /api/scheduler/run
```

#### 启动调度服务

```http
POST /api/scheduler/start
```

#### 停止调度服务

```http
POST /api/scheduler/stop
```

### 系统接口

#### 健康检查

```http
GET /health
```

#### API版本信息

```http
GET /api
```

## 命令行脚本

### 批量获取股票数据

```bash
npm run scrape 600519 000001 300750
```

### 更新所有缓存数据

```bash
npm run update
```

## 自动更新配置

### Cron表达式说明

调度任务使用cron表达式，格式如下：

```
分 时 日 月 周
```

**示例**：

- `0 2 * * *` - 每天凌晨2点执行
- `0 0 1 * *` - 每月1日凌晨执行
- `0 2 * * 1` - 每周一凌晨2点执行
- `0 2 1 * *` - 每月1日凌晨2点执行

### 启用自动更新

编辑 `.env` 文件：

```env
AUTO_UPDATE_ENABLED=true
SCHEDULE_CRON=0 2 * * *
```

然后重启服务。

## 数据存储

### 存储位置

- **数据目录**: `data/`
- **股票数据**: `data/stocks/{股票代码}.json`
- **观察列表**: `data/watchlist.json`
- **日志目录**: `logs/`
  - 错误日志: `logs/error.log`
  - 综合日志: `logs/combined.log`

### 缓存机制

- 缓存时长：24小时（可配置）
- 缓存位置：内存 + 文件
- 自动过期检查

## 前端集成

在前端应用中，可以通过以下方式调用API：

```typescript
// 获取股票数据
const fetchStockData = async (symbol: string) => {
  const response = await fetch(`http://localhost:3001/api/stocks/${symbol}?fetch=true`);
  const data = await response.json();
  return data;
};

// 搜索股票
const searchStocks = async (keyword: string) => {
  const response = await fetch(`http://localhost:3001/api/stocks/search/${keyword}`);
  const data = await response.json();
  return data;
};
```

### React集成示例

```tsx
import { useState, useEffect } from 'react';

interface StockData {
  symbol: string;
  name: string;
  historicalData: Array<{
    year: number;
    metrics: {
      revenue: number;
      netProfit: number;
      roe: number;
    };
  }>;
}

function StockAnalysis({ symbol }: { symbol: string }) {
  const [stockData, setStockData] = useState<StockData | null>(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      try {
        const response = await fetch(`http://localhost:3001/api/stocks/${symbol}?fetch=true`);
        const data = await response.json();
        if (data.success) {
          setStockData(data.data);
        }
      } catch (error) {
        console.error('获取数据失败:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [symbol]);

  if (loading) return <div>加载中...</div>;
  if (!stockData) return <div>暂无数据</div>;

  return (
    <div>
      <h1>{stockData.name} ({stockData.symbol})</h1>
      {/* 渲染分析图表 */}
    </div>
  );
}
```

## 故障排除

### 常见问题

#### 1. 请求超时

如果遇到请求超时错误，可以增加超时时间：

```env
THS_REQUEST_TIMEOUT=60000
```

#### 2. 被网站限制

如果被同花顺网站限制，可以增加请求间隔：

```env
THS_API_DELAY=5000
```

#### 3. 端口被占用

修改服务端口：

```env
PORT=3002
```

#### 4. 数据解析错误

检查日志文件：

```bash
tail -f logs/error.log
```

### 日志级别

可以调整日志级别来获取更多信息：

```env
LOG_LEVEL=debug  # 最详细
LOG_LEVEL=info   # 默认
LOG_LEVEL=warn   # 警告以上
LOG_LEVEL=error  # 仅错误
```

## 扩展功能

### 添加新的数据源

1. 在 `types/stock.ts` 中添加数据源配置
2. 创建新的爬虫服务
3. 在路由中注册新接口

### 自定义分析指标

在 `services/thsScraper.ts` 的 `calculateDerivedMetrics` 方法中添加自定义计算逻辑。

## 许可证

MIT License

## 贡献

欢迎提交Issue和Pull Request。
