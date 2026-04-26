# Design Decisions

## Architectural Decision Records (ADR)

---

### Decision 1: Dual Database Architecture

**Context:**
Financial trading systems require both time-series data (OHLCV, ticks) and relational metadata (symbols, news, earnings). A single database would either compromise on time-series performance or relational querying capabilities.

**Decision:**
Implement dual-database architecture:
- QuestDB for time-series data
- PostgreSQL for relational metadata

**Reasoning:**
- QuestDB optimized for high-frequency time-series ingestion and queries
- PostgreSQL provides robust relational features, ACID compliance, and complex queries
- Each database can be tuned for its specific workload
- Clear separation of concerns based on data characteristics

**Consequences:**
- ✅ Optimal performance for each data type
- ✅ Scalability: databases can be scaled independently
- ✅ Specialized query patterns per database
- ⚠️ Increased operational complexity (two databases to manage)
- ⚠️ Need for router logic to direct queries to correct database
- ⚠️ Cross-database joins require application-level logic

---

### Decision 2: Async-First Architecture

**Context:**
Financial data systems involve heavy I/O: API calls, database operations, and concurrent pipeline execution. Blocking operations would severely limit throughput.

**Decision:**
Use asyncio throughout the stack: aiohttp for HTTP, asyncpg for PostgreSQL, async FastAPI endpoints, and async pipeline operations.

**Reasoning:**
- Maximize concurrency for I/O-bound operations
- Single-threaded async model reduces complexity vs threading
- FastAPI natively supports async patterns
- asyncpg provides superior PostgreSQL performance
- Enables thousands of concurrent connections with low overhead

**Consequences:**
- ✅ High throughput for concurrent operations
- ✅ Efficient resource utilization
- ✅ Natural fit with FastAPI ecosystem
- ⚠️ Requires async discipline (no blocking calls in async context)
- ⚠️ More complex error handling and debugging
- ⚠️ Learning curve for developers unfamiliar with async/await

---

### Decision 3: No ORM - Raw SQL Schema Management

**Context:**
Traditional ORMs and migration tools (Alembic, Django migrations) add abstraction layers. Financial systems require precise schema control and performance optimization.

**Decision:**
Use raw SQL scripts in `scripts/` directory for schema management. No SQLAlchemy ORM or migration frameworks.

**Reasoning:**
- Direct control over SQL for performance optimization
- Simpler mental model: what you write is what runs
- No ORM impedance mismatch
- Easier to optimize queries for financial data patterns
- Schema changes are explicit and reviewable as SQL

**Consequences:**
- ✅ Full SQL control and optimization
- ✅ No hidden ORM queries or N+1 problems
- ✅ Explicit, auditable schema changes
- ⚠️ Manual schema versioning and migration tracking
- ⚠️ More boilerplate for CRUD operations
- ⚠️ Requires SQL expertise from developers

---

### Decision 4: Modular Sentiment Analysis System

**Context:**
Previous monolithic sentiment systems were difficult to maintain and extend. LangChain/LangGraph provide modern patterns for LLM-based analysis.

**Decision:**
Implement modular sentiment analysis in `data_engineering/sentiment_analysis/` using LangChain and LangGraph for orchestration.

**Reasoning:**
- LangChain provides standardized LLM interfaces
- LangGraph enables complex multi-step analysis workflows
- Modular design allows independent testing and updates
- Easier to swap LLM providers or add new analysis types
- Industry-standard tools with active community

**Consequences:**
- ✅ Flexible, extensible sentiment pipeline
- ✅ Easy to test individual components
- ✅ Can leverage LangChain ecosystem (agents, tools, memory)
- ⚠️ Additional dependency on LangChain/LangGraph
- ⚠️ Learning curve for LangGraph state machines
- ⚠️ Potential overhead vs direct API calls

---

### Decision 5: Type Safety with Python 3.11+ Syntax

**Context:**
Financial systems have complex data structures. Type errors in production can lead to incorrect calculations or failed trades.

**Decision:**
Enforce strict type hints using modern Python 3.11+ syntax (`list[str]` instead of `List[str]`, `X | None` instead of `Optional[X]`).

**Reasoning:**
- Catch type errors at development time
- Improved IDE autocomplete and refactoring
- Self-documenting code through types
- Modern syntax is cleaner and more readable
- Python 3.11+ performance improvements

**Consequences:**
- ✅ Fewer runtime type errors
- ✅ Better developer experience with IDE support
- ✅ Code serves as documentation
- ⚠️ Requires Python 3.11+ (no backward compatibility)
- ⚠️ Initial overhead to add types to all code
- ⚠️ Type checking discipline required from all developers

---

### Decision 6: APScheduler for Pipeline Automation

**Context:**
Financial data needs regular updates: minute bars, daily OHLCV, news feeds. System needs reliable job scheduling.

**Decision:**
Use APScheduler for all pipeline scheduling needs in `pipeline/schedulers/`.

**Reasoning:**
- Python-native scheduling library
- Supports cron-like expressions and interval-based jobs
- Persistent job stores available
- Integrates well with async code
- No external dependencies like Celery/Redis needed for basic use

**Consequences:**
- ✅ Simple, lightweight scheduling
- ✅ Good for development and small-to-medium deployments
- ✅ Easy to debug within same process
- ⚠️ Single-process limitation (no distributed scheduling)
- ⚠️ Jobs stop if application stops (unless using persistent store)
- ⚠️ May need upgrade to Celery/Airflow for production scale

---

## Future Decisions to Track

- Caching strategy (Redis vs in-memory)
- Authentication/authorization for API
- Data retention and archival policies
- Monitoring and observability tools
- Production deployment strategy (Docker, Kubernetes, etc.)
- API rate limiting and throttling approach
