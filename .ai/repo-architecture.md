# System Architecture

## Core Components

### 1. **Dual Database System**
   - **QuestDB (Time-Series)**: High-performance storage for OHLCV data, tick data, and metrics
   - **PostgreSQL (Relational)**: Metadata storage for symbols, companies, news, earnings

### 2. **API Layer**
   - **FastAPI**: Async REST API for data access and system control
   - **Routers**: Modular endpoint organization
   - **Entry Point**: `api/main.py`

### 3. **Data Sources**
   - **Schwab API**: Primary brokerage data
   - **Yahoo Finance**: Supplementary market data
   - **News APIs**: Sentiment and fundamental data sources

### 4. **Pipeline System**
   - **Data Ingestion**: ETL pipelines for market data
   - **Schedulers**: APScheduler-based automation
   - **Processing**: Transform → Store workflow

### 5. **Data Engineering**
   - **Feature Engineering**: Technical indicators (pandas-ta, ta-lib)
   - **Sentiment Analysis**: Modular system using LangChain/LangGraph

### 6. **Configuration**
   - **Pydantic Settings**: Centralized config in `config/settings.py`
   - **Environment Variables**: Secure credential management

## Data Flow

```
External APIs (Schwab/Yahoo/News)
    ↓
Data Sources Layer
    ↓
Transformation & Feature Engineering
    ↓
Storage Router (Time-Series vs Relational)
    ↓
    ├─→ QuestDB (OHLCV, Ticks, Metrics)
    └─→ PostgreSQL (Symbols, News, Earnings)
    ↓
FastAPI Endpoints
    ↓
Clients/Consumers
```

## Key Interfaces

### Storage Clients
- `database/storage/questdb_client.py` - ILP protocol for time-series writes
- `database/storage/postgres_client.py` - asyncpg for relational operations

### API Contracts
- REST endpoints exposed via FastAPI routers
- Async request/response pattern throughout

### Pipeline Contracts
- Source → Transform → Store pattern
- Error handling prevents cascade failures

## External Integrations

### Databases
- **QuestDB**: ILP port (default 9009), HTTP API
- **PostgreSQL**: asyncpg connection pool

### APIs
- **Schwab API**: OAuth2 authentication
- **Yahoo Finance**: Public endpoints
- **News/Sentiment Services**: API key authentication

### AI/ML Services
- **LangChain**: LLM orchestration for sentiment
- **LangGraph**: Agent workflow system
- **pandas-ta / ta-lib**: Technical analysis libraries

## Technology Stack

### Core Languages
- **Python 3.11+**: Modern syntax with strict typing

### Web Framework
- **FastAPI**: Async web framework
- **Uvicorn**: ASGI server

### Database
- **QuestDB**: Time-series database
- **PostgreSQL**: Relational database
- **asyncpg**: Async PostgreSQL driver

### Data Processing
- **pandas**: Data manipulation
- **pandas-ta**: Technical indicators
- **ta-lib**: Advanced technical analysis

### AI/LLM
- **LangChain**: LLM framework
- **LangGraph**: Agent orchestration

### Infrastructure
- **Docker**: Containerization
- **APScheduler**: Job scheduling
- **Pydantic**: Settings and validation

### HTTP
- **aiohttp**: Async HTTP client

### Testing
- **pytest**: Test framework
- **pytest markers**: Test categorization

## Architectural Principles

1. **Async-First**: All I/O operations use asyncio
2. **Type Safety**: Strict type hints (Python 3.11+ syntax)
3. **Separation of Concerns**: Clear boundaries between layers
4. **Database Specialization**: Right tool for the job (time-series vs relational)
5. **Modular Pipelines**: Independent, composable data flows
6. **Error Isolation**: Pipeline failures don't cascade
7. **Configuration Centralization**: Single source of truth in settings
8. **No ORM Migrations**: Raw SQL for schema management
