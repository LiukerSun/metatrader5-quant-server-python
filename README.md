# MetaTrader5 Quantitative Trading Server

基于 Python 的 MetaTrader5 量化交易服务器，提供自动化算法交易、REST API 接口和企业级监控能力。

## 功能特性

- **自动化交易**: 内置均值回归策略，支持可配置参数
- **风险管理**: 16 级追踪止损机制
- **多资产支持**: 外汇、大宗商品、贵金属、原油等
- **实时监控**: Grafana 仪表盘 + Prometheus 指标
- **可扩展架构**: Celery 分布式任务处理
- **API 优先设计**: RESTful API + Swagger 文档
- **容器化部署**: 完整的 Docker Compose 编排
- **安全通信**: Traefik + Let's Encrypt HTTPS
- **日志聚合**: Loki + Grafana 日志分析

## 技术栈

| 组件 | 技术 |
|------|------|
| MT5 API 服务 | Flask, Flasgger |
| 量化引擎 | Django 4.2, Django REST Framework |
| 任务队列 | Celery 5.4, Redis 6 |
| 数据库 | PostgreSQL 15 |
| 数据处理 | Pandas, NumPy |
| 反向代理 | Traefik 3.6 |
| 监控 | Grafana 11, Prometheus, Loki 3.0 |
| 容器化 | Docker, Docker Compose |

## 项目结构

```
├── backend/
│   ├── mt5/                    # Flask MT5 API 服务
│   │   ├── app/
│   │   │   ├── routes/         # API 路由
│   │   │   └── app.py          # 应用入口
│   │   └── Dockerfile
│   │
│   └── django/                 # Django 量化交易引擎
│       ├── app/
│       │   ├── quant/          # 量化交易模块
│       │   │   ├── algorithms/ # 交易算法
│       │   │   ├── indicators/ # 技术指标
│       │   │   └── tasks.py    # Celery 任务
│       │   ├── nexus/          # 数据聚合 API
│       │   └── utils/          # 工具模块
│       └── Dockerfile
│
├── monitoring/                 # 监控配置
│   ├── configs/                # 各组件配置
│   └── dashboards/             # Grafana 仪表盘
│
├── docker-compose.yml          # 服务编排
└── .env.example                # 环境变量模板
```

## 快速开始

### 前置要求

- Docker & Docker Compose
- MetaTrader5 账户凭证

### 安装步骤

1. **克隆仓库**
```bash
git clone <repository-url>
cd metatrader5-quant-server-python
```

2. **配置环境变量**
```bash
cp .env.example .env
# 编辑 .env 文件，填入你的配置
```

3. **启动服务**
```bash
docker-compose up -d
```

4. **验证部署**
```bash
# 检查服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

## 环境变量配置

```bash
# MT5 后端
CUSTOM_USER=admin
PASSWORD=your_password
VNC_DOMAIN=vnc.mt5.example.com
API_DOMAIN=api.mt5.example.com
MT5_API_PORT=5001

# Django 后端
DJANGO_DOMAIN=django.mt5.example.com
MT5_API_URL=http://mt5:5001

# 数据库
POSTGRES_DB=postgres
POSTGRES_USER=admin
POSTGRES_PASSWORD=your_db_password

# Celery
CELERY_BROKER_URL=redis://redis:6379/0
CELERY_RESULT_BACKEND=redis://redis:6379/0

# 监控
GRAFANA_DOMAIN=grafana.mt5.example.com
```

## API 端点

### MT5 API (Flask)

| 端点 | 方法 | 描述 |
|------|------|------|
| `/health` | GET | 健康检查 |
| `/order` | POST | 下单 |
| `/position` | GET/DELETE | 持仓管理 |
| `/symbol` | GET | 交易品种信息 |
| `/fetch_data_pos` | GET | 获取 OHLC 数据 |
| `/history` | GET | 交易历史 |

访问 `http://api.mt5.example.com/apidocs` 查看完整的 Swagger 文档。

## 交易算法

### 均值回归策略

配置参数 (`backend/django/app/quant/algorithms/mean_reversion/config.py`):

- **交易品种**: NG, BRN, WTI, XAGUSD, XAUUSD, EURUSD 等
- **时间周期**: M15 (15分钟)
- **杠杆**: 200x
- **单笔资金**: $100
- **追踪止损**: 16 级，从 4.00x 到 0.025x 盈亏倍数

### Celery 定时任务

- `run_quant_entry_algorithm` - 入场信号检测
- `run_quant_trailing_stop_algorithm` - 追踪止损管理
- `run_quant_close_algorithm` - 平仓逻辑

## 监控服务

| 服务 | 端口 | 描述 |
|------|------|------|
| Grafana | 3000 | 可视化仪表盘 |
| Prometheus | 9090 | 指标数据库 |
| Loki | 3100 | 日志聚合 |
| AlertManager | 9093 | 告警管理 |

## 开发指南

### 本地开发

```bash
# 安装 MT5 API 依赖
cd backend/mt5/app
pip install -r requirements.txt

# 安装 Django 依赖
cd backend/django
pip install -r requirements.txt

# 运行 Django 开发服务器
python manage.py runserver

# 运行 Celery Worker
celery -A app worker --loglevel=info

# 运行 Celery Beat
celery -A app beat --loglevel=info
```

### 添加新算法

1. 在 `backend/django/app/quant/algorithms/` 创建新目录
2. 实现 `config.py`, `entry.py`, `trailing.py` 等模块
3. 在 `tasks.py` 中注册 Celery 任务
4. 配置 Celery Beat 调度

## 许可证

MIT License

## 风险提示

本项目仅供学习和研究目的。量化交易存在风险，请谨慎使用。作者不对任何交易损失承担责任。
