# Code Map

## Directory Structure Overview

This document provides a high-level index of the codebase organization, making it easy to navigate and understand where different components live.

---

## Root Level

```
.ai/                           # Repository memory system (this directory)
.github/                       # GitHub configuration
  └── copilot-instructions.md  # AI coding assistant instructions
api/                           # FastAPI application and REST endpoints
config/                        # Configuration and settings
data_sources/                  # External API integrations
database/                      # Database clients and schema
  ├── schema/                  # SQL schema definitions
  └── storage/                 # Database client implementations
data_engineering/              # Feature engineering and analytics
  ├── features/                # Technical indicators and features
  └── sentiment_analysis/      # Sentiment analysis modules
pipeline/                      # Data ingestion and processing pipelines
  └── schedulers/              # Job scheduling logic
scripts/                       # Maintenance and utility scripts
tests/                         # Test suite
```

---

## API Layer (`api/`)

**Purpose:** REST API built with FastAPI for external access to the system.

### Key Files
- `main.py` - FastAPI application entry point
- `routers/` - Modular endpoint definitions
  - Market data endpoints
  - Symbol/metadata endpoints
  - Pipeline control endpoints
  - Health/status endpoints

### Responsibilities
- HTTP request handling
- Authentication/authorization
- Request validation
- Response serialization
- Error handling and HTTP status codes

### Important Patterns
- All endpoints are async (`async def`)
- Use Pydantic models for request/response validation
- Router-based organization for scalability
- Dependency injection for database clients

---

## Configuration (`config/`)

**Purpose:** Centralized configuration management using Pydantic.

### Key Files
- `settings.py` - Main settings class with environment variables
- `logging_config.py` - Logging setup and configuration

### Responsibilities
- Environment variable loading
- Configuration validation
- Default values
- Secrets management (API keys, database credentials)

### Important Patterns
- No hardcoded credentials
- Type-safe settings with Pydantic
- Environment-specific overrides (.env files)

---

## Data Sources (`data_sources/`)

**Purpose:** Integrations with external data providers.

### Key Modules
- `schwab/` - Charles Schwab API integration
  - OAuth2 authentication
  - Market data fetching
  - Account data access
- `yahoo/` - Yahoo Finance integration
  - Historical data
  - Real-time quotes
- `news/` - News API integrations
  - News article fetching
  - Metadata extraction

### Responsibilities
- API authentication
- HTTP request handling
- Rate limiting
- Data transformation to internal format
- Error handling and retries

### Important Patterns
- Use `aiohttp` for async HTTP requests
- Implement exponential backoff for retries
- Transform external data to canonical internal format
- Handle API-specific quirks and limitations

---

## Database (`database/`)

**Purpose:** Dual database system for time-series and relational data.

### Structure

#### `database/schema/`
- `questdb_schema.sql` - QuestDB table definitions
- `postgres_schema.sql` - PostgreSQL schema (tables, indexes, constraints)

#### `database/storage/`
- `questdb_client.py` - QuestDB client using ILP protocol
- `postgres_client.py` - PostgreSQL client using asyncpg

### Responsibilities
- Database connections and pooling
- Query execution
- Transaction management
- Schema initialization

### Important Patterns
- **QuestDB**: Time-series data (OHLCV, ticks, metrics)
  - Use ILP (Influx Line Protocol) for high-speed inserts
  - Optimized for time-range queries
- **PostgreSQL**: Relational metadata (symbols, news, earnings)
  - Use asyncpg for async operations
  - Connection pooling for performance
- Raw SQL (no ORM) for full control
- Async all the way

---

## Data Engineering (`data_engineering/`)

**Purpose:** Feature extraction, technical analysis, and sentiment analysis.

### Structure

#### `data_engineering/features/`
- Technical indicator calculations
- Custom feature engineering
- Integration with pandas-ta and ta-lib

#### `data_engineering/sentiment_analysis/`
- LangChain/LangGraph-based sentiment system
- Modular analysis components
- LLM integration for news analysis

### Responsibilities
- Calculate technical indicators (SMA, RSI, MACD, etc.)
- Engineer custom features
- Sentiment scoring of news and social media
- Feature versioning and storage

### Important Patterns
- Use pandas for vectorized calculations
- Leverage pandas-ta/ta-lib for standard indicators
- LangChain for LLM orchestration
- Modular design for easy testing and extension

---

## Pipelines (`pipeline/`)

**Purpose:** Orchestrate data ingestion, transformation, and storage.

### Structure

#### `pipeline/`
- Individual pipeline implementations
- Data validation and cleaning logic

#### `pipeline/schedulers/`
- APScheduler configurations
- Job definitions and triggers
- Cron expressions for regular updates

### Responsibilities
- Fetch data from sources
- Transform to internal format
- Validate data quality
- Route to appropriate database (QuestDB vs PostgreSQL)
- Error handling and logging

### Important Patterns
- Pipeline flow: Source → Transform → Store
- Each pipeline is independently testable
- Errors don't cascade to other pipelines
- Idempotent operations where possible
- Comprehensive logging

---

## Scripts (`scripts/`)

**Purpose:** Maintenance, setup, and utility operations.

### Key Scripts
- `recreate_postgres_schema.py` - Drop and recreate PostgreSQL schema
- `clear_database_data.py` - Clear data from databases
- `populate_sample_tickers.py` - Seed database with sample symbols
- Database backup/restore utilities
- Data quality validation scripts

### Responsibilities
- Database maintenance
- Development setup
- Data migration
- Testing utilities

### Important Patterns
- Run standalone (not part of main application)
- Idempotent where possible
- Clear logging of operations
- Require confirmation for destructive operations

---

## Tests (`tests/`)

**Purpose:** Comprehensive test suite for all components.

### Structure
- Unit tests for individual functions/classes
- Integration tests for pipelines and databases
- Test fixtures and mocks
- Performance/load tests

### Responsibilities
- Verify correctness
- Catch regressions
- Document expected behavior
- Enable safe refactoring

### Important Patterns
- Use pytest framework
- Markers for test categorization (`@pytest.mark.slow`)
- Mock external APIs in unit tests
- Use Docker containers for integration tests
- Keep tests fast (avoid slow tests in CI)

---

## Key Cross-Cutting Concerns

### Async/Await
- **Used in:** All I/O operations (API, database, HTTP)
- **Pattern:** Never block in async context

### Type Hints
- **Used in:** All Python code
- **Pattern:** Python 3.11+ syntax, strict typing

### Error Handling
- **Used in:** Especially in pipelines and API endpoints
- **Pattern:** Try/except with logging, graceful degradation

### Logging
- **Used in:** Throughout the codebase
- **Configuration:** `config/logging_config.py`

### Configuration
- **Used in:** All modules requiring config
- **Pattern:** Import from `config/settings.py`, no hardcoded values

---

## Navigation Tips

1. **Starting point for new features:** Usually begins in `data_sources/` or `pipeline/`
2. **API changes:** Start in `api/routers/`
3. **Database schema:** Look in `database/schema/`
4. **Configuration changes:** Always in `config/settings.py`
5. **Maintenance tasks:** Check `scripts/`
6. **Understanding data flow:** Trace from `data_sources/` → `pipeline/` → `database/storage/`

---

## Common Workflows

### Adding a New Data Source
1. Create module in `data_sources/`
2. Implement fetcher with async methods
3. Add to configuration in `config/settings.py`
4. Create pipeline in `pipeline/`
5. Add scheduler in `pipeline/schedulers/`
6. Write tests

### Adding a New API Endpoint
1. Create router in `api/routers/`
2. Define Pydantic models for request/response
3. Implement async handler
4. Add router to `api/main.py`
5. Write integration tests

### Adding a New Feature
1. Implement calculation in `data_engineering/features/`
2. Create pipeline to calculate and store
3. Add tests for calculation logic
4. Optionally expose via API

### Database Schema Change
1. Modify SQL in `database/schema/`
2. Update relevant client methods in `database/storage/`
3. Create migration script in `scripts/` if needed
4. Update tests
5. Document in `.ai/design-decisions.md` if significant
