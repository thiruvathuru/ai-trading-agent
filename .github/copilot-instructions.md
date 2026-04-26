## 💡 Code Style Preferences

- Prioritize **consistent, elegant, and precise** code that is easy to read and maintain.
- Avoid overly elaborate or verbose implementations and documentation.
- Write code that clearly expresses intent with minimal complexity.
- Let simplicity guide design choices, balancing clarity and functionality.
- When generating or modifying code, always consider these principles as primary requirements.


# AI Trader Data Services - AI Coding Instructions

You are an expert Python Data Engineer working on a high-performance financial data system.
This project uses a dual-database architecture: **QuestDB** for time-series data and **PostgreSQL** for relational metadata.

## 🏗️ Architecture & Patterns

- **Dual Storage Strategy**:
  - **QuestDB (Time-Series)**: Use for OHLCV, tick data, and high-frequency metrics. Interact via ILP (Influx Line Protocol) using `database/storage/questdb_client.py`.
  - **PostgreSQL (Metadata)**: Use for symbols, company profiles, news, and earnings. Interact via `asyncpg` using `database/storage/postgres_client.py`.
  - **Pattern**: Always choose the right store based on data nature (Time-series vs. Relational).

- **Async-First Design**:
  - Use `asyncio` and `aiohttp` for all I/O operations.
  - Database clients (`PostgresClient`, `QuestDBClient`) are async.
  - API endpoints (`FastAPI`) must be `async def`.

- **Modular Pipelines**:
  - Pipelines reside in `pipeline/`.
  - Schedulers (`pipeline/schedulers/`) use `APScheduler` to trigger pipelines.
  - Data flow: `Source -> Transformation -> Storage`.

- **Data Engineering**:
  - **Features**: Use `pandas-ta` and `ta-lib` in `data_engineering/features/`.
  - **Sentiment**: New modular system in `data_engineering/sentiment_analysis/` using LangChain/LangGraph.

## 🛠️ Development Workflows

- **Dependency Management**:
  - Add new dependencies to `requirements.txt`.
  - Core libs: `fastapi`, `asyncpg`, `questdb`, `pandas`, `langchain`.

- **Database Management**:
  - **Schema Changes**: Do not use ORM migrations. Use raw SQL scripts in `scripts/` (e.g., `recreate_postgres_schema.py`).
  - **QuestDB**: Schema is often implicit via ILP, but explicit DDL is in `database/schema/questdb_schema.sql`.
  - **Scripts**: Use `scripts/` for maintenance (e.g., `clear_database_data.py`, `populate_sample_tickers.py`).

- **Testing**:
  - Run tests: `pytest`
  - Markers: `pytest -m "not slow"` for quick feedback.
  - Integration tests require running Docker containers.

- **Running the System**:
  - API: `uvicorn api.main:app --reload`
  - Demo/Pipeline: `python main.py`

## 📝 Coding Conventions

- **Type Hinting**: Enforce strict typing. Use Python 3.11+ syntax (e.g., `list[str] | None` instead of `List[str]`, `Optional[str]`).
- **Configuration**: Use `config/settings.py` (Pydantic) for all config. Do not hardcode credentials.
- **Error Handling**:
  - Use `try/except` blocks in data fetchers to prevent pipeline crashes.
  - Log errors using `config.logging_config`.
- **Docstrings**: Use Google-style docstrings for all functions and classes.

## 📂 Key Directories

- `api/`: FastAPI routers and entry point.
- `config/`: Centralized configuration and settings.
- `data_sources/`: External API integrations (Schwab, Yahoo, etc.).
- `database/`: Storage clients and schema definitions.
- `pipeline/`: Business logic for data ingestion and processing.
- `scripts/`: Utilities for DB maintenance and data population.
