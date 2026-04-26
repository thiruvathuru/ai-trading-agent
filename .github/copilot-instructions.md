# AI Trading Agent - Copilot Instructions

> **Expert Python Data Engineer** working on a high-performance financial data system with dual-database architecture.

---

## 🧠 CRITICAL: Repository Memory System

**BEFORE writing ANY code, ALWAYS load context from `.ai/` directory:**

```
1. .ai/session-context.md      → Current session state
2. .ai/repo-architecture.md    → System architecture
3. .ai/git-history-context.md  → Recent commit history
4. .ai/design-decisions.md     → Architectural decisions (ADRs)
5. .ai/code-map.md             → Directory structure & navigation
6. .ai/task-tracker.md         → Development roadmap
```

**After making changes, UPDATE:**
- `.ai/session-context.md` - Document files modified and decisions
- `.ai/git-history-context.md` - After commits
- `.ai/design-decisions.md` - For architectural changes
- `.ai/task-tracker.md` - Update task status

---

## 🏗️ Architecture Overview

### Dual Database System (CRITICAL PATTERN)

**QuestDB (Time-Series):**
- **Use for:** OHLCV data, tick data, high-frequency metrics, time-series analytics
- **Client:** `database/storage/questdb_client.py`
- **Protocol:** ILP (Influx Line Protocol) for writes
- **Pattern:** Optimized for time-range queries and high-speed ingestion

**PostgreSQL (Relational):**
- **Use for:** Symbols, company profiles, news articles, earnings, metadata
- **Client:** `database/storage/postgres_client.py`
- **Driver:** `asyncpg` (async connection pool)
- **Pattern:** Complex queries, joins, ACID transactions

**Decision Rule:** Is it timestamped, high-frequency data? → QuestDB. Is it relational metadata? → PostgreSQL.

---

## 💻 Code Style & Standards

### Type Safety (MANDATORY)

```python
# ✅ CORRECT - Python 3.11+ syntax
def fetch_ohlcv(symbol: str, start: datetime) -> list[dict[str, Any]]:
    result: dict[str, float] | None = get_price(symbol)
    
# ❌ WRONG - Old syntax
from typing import List, Optional, Dict
def fetch_ohlcv(symbol: str) -> List[Dict[str, Any]]:
    result: Optional[Dict[str, float]] = get_price(symbol)
```

**Requirements:**
- Use Python 3.11+ type syntax exclusively
- Strict type hints on all functions, methods, and variables
- No `typing.List`, `typing.Dict`, `typing.Optional` imports
- Use `X | None` instead of `Optional[X]`

### Async-First Design (MANDATORY)

```python
# ✅ CORRECT - Async all the way
async def fetch_market_data(symbol: str) -> dict[str, Any]:
    async with aiohttp.ClientSession() as session:
        async with session.get(f"/api/{symbol}") as response:
            return await response.json()

# ❌ WRONG - Blocking in async context
async def fetch_market_data(symbol: str) -> dict[str, Any]:
    response = requests.get(f"/api/{symbol}")  # NEVER DO THIS
    return response.json()
```

**Requirements:**
- All I/O operations MUST be async (database, HTTP, file I/O)
- Use `asyncio`, `aiohttp`, `asyncpg`
- FastAPI endpoints: `async def`
- Never use blocking libraries (requests, urllib, time.sleep) in async context

### Database Patterns (MANDATORY)

```python
# ✅ CORRECT - Raw SQL with type safety
async def get_symbol_info(symbol: str) -> dict[str, Any] | None:
    query = """
        SELECT symbol, name, exchange, sector
        FROM symbols
        WHERE symbol = $1
    """
    async with postgres_client.pool.acquire() as conn:
        row = await conn.fetchrow(query, symbol)
        return dict(row) if row else None

# ❌ WRONG - No ORM allowed
from sqlalchemy import select
result = session.execute(select(Symbol).where(Symbol.symbol == symbol))
```

**Requirements:**
- **NO ORM** - Use raw SQL for PostgreSQL
- **NO Migration Tools** - Use SQL scripts in `scripts/`
- Use parameterized queries ($1, $2) to prevent SQL injection
- Schema changes via `database/schema/*.sql` files

### Error Handling

```python
# ✅ CORRECT - Graceful degradation in pipelines
async def run_pipeline(symbol: str) -> None:
    try:
        data = await fetch_market_data(symbol)
        await store_to_questdb(data)
        logger.info(f"Pipeline success: {symbol}")
    except aiohttp.ClientError as e:
        logger.error(f"API error for {symbol}: {e}")
        # Don't crash, continue with other symbols
    except Exception as e:
        logger.exception(f"Unexpected error in pipeline: {e}")

# ❌ WRONG - Let errors crash the pipeline
async def run_pipeline(symbol: str) -> None:
    data = await fetch_market_data(symbol)  # Crashes on API error
    await store_to_questdb(data)
```

### Configuration (MANDATORY)

```python
# ✅ CORRECT - Use centralized config
from config.settings import settings

db_url = settings.postgres_url
api_key = settings.schwab_api_key

# ❌ WRONG - Hardcoded values
db_url = "postgresql://localhost:5432/trading"
api_key = "abc123xyz"  # NEVER DO THIS
```

**Requirements:**
- All config in `config/settings.py` (Pydantic)
- Use environment variables via `.env` file
- Never hardcode credentials, URLs, or API keys

### Docstrings (Google Style)

```python
async def fetch_ohlcv(
    symbol: str,
    start_date: datetime,
    end_date: datetime,
    interval: str = "1d"
) -> list[dict[str, Any]]:
    """Fetch OHLCV data for a symbol within a date range.
    
    Args:
        symbol: Stock ticker symbol (e.g., "AAPL")
        start_date: Start of date range (inclusive)
        end_date: End of date range (inclusive)
        interval: Candle interval ("1m", "5m", "1h", "1d")
        
    Returns:
        List of OHLCV dictionaries with keys:
        - timestamp: datetime
        - open, high, low, close: float
        - volume: int
        
    Raises:
        ValueError: If start_date > end_date
        aiohttp.ClientError: If API request fails
    """
```

---

## 📂 Project Structure & Navigation

```
api/                           # FastAPI REST API
  ├── main.py                  # App entry point
  └── routers/                 # Endpoint modules

config/                        # Configuration
  ├── settings.py              # Pydantic settings (SINGLE SOURCE OF TRUTH)
  └── logging_config.py        # Logging setup

data_sources/                  # External API integrations
  ├── schwab/                  # Schwab API client
  ├── yahoo/                   # Yahoo Finance client
  └── news/                    # News API clients

database/                      # Database layer
  ├── schema/                  # SQL schema definitions
  │   ├── questdb_schema.sql   # QuestDB tables
  │   └── postgres_schema.sql  # PostgreSQL schema
  └── storage/                 # Database clients
      ├── questdb_client.py    # QuestDB ILP client
      └── postgres_client.py   # PostgreSQL asyncpg client

data_engineering/              # Feature engineering & analytics
  ├── features/                # Technical indicators (pandas-ta, ta-lib)
  └── sentiment_analysis/      # LangChain/LangGraph sentiment

pipeline/                      # Data pipelines
  ├── schedulers/              # APScheduler job definitions
  └── *.py                     # Pipeline implementations

scripts/                       # Maintenance utilities
  ├── recreate_postgres_schema.py
  ├── clear_database_data.py
  └── populate_sample_tickers.py

tests/                         # Test suite
  ├── unit/                    # Unit tests
  └── integration/             # Integration tests (require Docker)
```

---

## 🔄 Common Workflows

### Adding a New Data Source

1. Create module in `data_sources/<provider>/`
2. Implement async fetcher with error handling
3. Add credentials to `config/settings.py`
4. Create pipeline in `pipeline/`
5. Add APScheduler job in `pipeline/schedulers/`
6. Write tests in `tests/`
7. Update `.ai/session-context.md` and `.ai/code-map.md`

### Adding a New API Endpoint

1. Create router in `api/routers/<resource>.py`
2. Define Pydantic models for request/response
3. Implement `async def` handler
4. Register router in `api/main.py`
5. Add integration tests
6. Update `.ai/session-context.md`

### Database Schema Change

1. Modify SQL in `database/schema/`
2. Update client methods in `database/storage/`
3. Create migration script in `scripts/` (if needed)
4. Test with `pytest tests/integration/`
5. Document in `.ai/design-decisions.md` if significant

### Adding Technical Indicators

1. Implement in `data_engineering/features/`
2. Use pandas-ta or ta-lib for standard indicators
3. Create feature calculation pipeline
4. Store results in QuestDB (time-series)
5. Add tests with sample data
6. Update `.ai/task-tracker.md`

---

## ⚡ Performance Patterns

### QuestDB Optimization

```python
# ✅ CORRECT - Batch inserts with ILP
from questdb.ingress import Sender

async def bulk_insert_ohlcv(data: list[dict]) -> None:
    with Sender('localhost', 9009) as sender:
        for row in data:
            sender.row(
                'ohlcv',
                symbols={'symbol': row['symbol']},
                columns={
                    'open': row['open'],
                    'high': row['high'],
                    'low': row['low'],
                    'close': row['close'],
                    'volume': row['volume']
                },
                at=row['timestamp']
            )
        sender.flush()

# ❌ WRONG - Individual inserts
for row in data:
    await execute_sql(f"INSERT INTO ohlcv VALUES (...)")  # Slow
```

### Connection Pooling

```python
# ✅ CORRECT - Reuse connection pool
postgres_client = PostgresClient()  # Initialize once
await postgres_client.connect()  # Create pool

async def query_data():
    async with postgres_client.pool.acquire() as conn:
        return await conn.fetch("SELECT * FROM symbols")

# ❌ WRONG - New connection every time
async def query_data():
    conn = await asyncpg.connect(DATABASE_URL)  # Expensive
    result = await conn.fetch("SELECT * FROM symbols")
    await conn.close()
    return result
```

---

## 🧪 Testing

### Unit Tests

```python
# tests/unit/test_data_sources.py
import pytest
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_fetch_market_data():
    """Test market data fetching with mocked API."""
    with patch('aiohttp.ClientSession.get') as mock_get:
        mock_get.return_value.__aenter__.return_value.json = AsyncMock(
            return_value={'price': 150.0}
        )
        
        result = await fetch_market_data('AAPL')
        
        assert result['price'] == 150.0
```

### Integration Tests

```python
# tests/integration/test_postgres.py
import pytest

@pytest.mark.integration  # Mark as integration test
@pytest.mark.asyncio
async def test_postgres_insert(postgres_client):
    """Test actual database insertion."""
    await postgres_client.execute(
        "INSERT INTO symbols (symbol, name) VALUES ($1, $2)",
        "TEST", "Test Company"
    )
    
    result = await postgres_client.fetchrow(
        "SELECT * FROM symbols WHERE symbol = $1", "TEST"
    )
    
    assert result['name'] == "Test Company"
```

**Running Tests:**
```bash
# Fast tests only (unit tests)
pytest -m "not slow"

# All tests including integration
pytest

# Specific test file
pytest tests/unit/test_data_sources.py

# With coverage
pytest --cov=. --cov-report=html
```

---

## 🚫 Anti-Patterns (DO NOT USE)

### ❌ NEVER Use ORMs
```python
# FORBIDDEN
from sqlalchemy import create_engine
from sqlalchemy.orm import Session
```

### ❌ NEVER Block in Async
```python
# FORBIDDEN
import requests
async def fetch():
    response = requests.get(url)  # Blocks event loop
```

### ❌ NEVER Hardcode Credentials
```python
# FORBIDDEN
API_KEY = "sk_live_abc123xyz"
DATABASE_URL = "postgresql://user:pass@localhost/db"
```

### ❌ NEVER Use Old Type Syntax
```python
# FORBIDDEN
from typing import List, Optional, Dict
def process(data: Optional[List[Dict]]) -> None:
```

### ❌ NEVER Ignore Errors in Pipelines
```python
# FORBIDDEN
async def pipeline():
    data = await fetch()  # Crashes entire pipeline if it fails
    await store(data)
```

---

## 🎯 Code Review Checklist

Before committing code, verify:

- [ ] All functions have type hints (Python 3.11+ syntax)
- [ ] All I/O operations are async
- [ ] Used correct database (QuestDB vs PostgreSQL)
- [ ] Configuration loaded from `config/settings.py`
- [ ] Error handling in place (especially in pipelines)
- [ ] Docstrings added (Google style)
- [ ] Tests written (unit + integration if needed)
- [ ] No hardcoded credentials or URLs
- [ ] No blocking calls in async context
- [ ] No ORM usage
- [ ] Updated `.ai/session-context.md`
- [ ] Updated `.ai/task-tracker.md` if task complete

---

## 📚 Key Libraries

**Core:**
- `fastapi` - Async web framework
- `asyncpg` - Async PostgreSQL driver
- `questdb` - QuestDB client (ILP)
- `aiohttp` - Async HTTP client
- `pandas` - Data manipulation
- `pydantic` - Settings and validation

**Data Engineering:**
- `pandas-ta` - Technical indicators
- `ta-lib` - Advanced technical analysis
- `langchain` - LLM orchestration
- `langgraph` - Agent workflows

**Infrastructure:**
- `apscheduler` - Job scheduling
- `uvicorn` - ASGI server
- `pytest` - Testing framework

---

## 🔍 Debugging Tips

### Check Database Connections
```bash
# PostgreSQL
psql -U postgres -h localhost -p 5432

# QuestDB
curl http://localhost:9000/exec?query=SELECT%20*%20FROM%20ohlcv%20LIMIT%2010
```

### View Logs
```python
# config/logging_config.py sets up logging
import logging
logger = logging.getLogger(__name__)

logger.debug("Detailed debug info")
logger.info("General info")
logger.error("Error occurred", exc_info=True)
```

### Profile Performance
```python
import asyncio
import time

async def timed_operation():
    start = time.perf_counter()
    await some_async_operation()
    elapsed = time.perf_counter() - start
    logger.info(f"Operation took {elapsed:.2f}s")
```

---

## 🎓 Learning Resources

**Internal Docs:**
- `.ai/repo-architecture.md` - System design
- `.ai/design-decisions.md` - Why things are built this way
- `.ai/code-map.md` - Where to find things
- `database/schema/*.sql` - Database schemas

**External:**
- [FastAPI Async](https://fastapi.tiangolo.com/async/)
- [asyncpg Documentation](https://magicstack.github.io/asyncpg/)
- [QuestDB ILP Guide](https://questdb.io/docs/reference/api/ilp/overview/)
- [pandas-ta Indicators](https://github.com/twopirllc/pandas-ta)

---

## 🚀 Running the System

### Development
```bash
# Start databases (Docker)
docker-compose up -d

# Run API server
uvicorn api.main:app --reload

# Run specific pipeline
python -m pipeline.ohlcv_pipeline

# Run tests
pytest
```

### Environment Setup
```bash
# Copy environment template
cp .env.example .env

# Edit with your credentials
# Never commit .env to git

# Install dependencies
pip install -r requirements.txt
```

---

## 🌟 Philosophy

**Elegant, Precise, Consistent:**
- Prioritize clarity over cleverness
- Let simplicity guide design
- Write code that expresses intent clearly
- Minimize complexity while maintaining functionality
- Every line should have a clear purpose

**Data Engineering Excellence:**
- Choose the right database for the data type
- Optimize for time-series workloads
- Handle errors gracefully (data pipelines fail)
- Log everything for debugging
- Test with real-world scenarios

**Performance Matters:**
- Async for I/O-bound operations
- Batch operations when possible
- Connection pooling
- Query optimization
- Profile before optimizing

---

**Remember:** Load `.ai/` context BEFORE coding. Update `.ai/` AFTER changes. Follow the architecture. Write async code. Use type hints. Test thoroughly. Keep it elegant.

---

*Last Updated: 2026-04-25*
*Version: 1.0*