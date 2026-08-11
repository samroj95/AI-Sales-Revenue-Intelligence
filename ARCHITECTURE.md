# Architecture

## Layered backend design

```
API layer (FastAPI routers)  →  thin: request/response only
   ↓
Repository layer              →  data-access abstraction (SOLID: DIP)
   ↓
SQLAlchemy models             →  ORM, single source of schema truth

Cross-cutting:
  app/core     — config, security (JWT/bcrypt), logging
  app/etl      — ingestion, cleaning, feature engineering, synthetic data
  app/ml       — forecasting (4 models), anomaly detection
  app/ai       — RAG chatbot (retrieval + LLM provider factory)
  app/alerts   — threshold rules → email/Slack/AI-recommendation
```

Each layer only knows about the layer directly below it. Endpoints never
issue raw SQL or touch SQLAlchemy `Session.query()` directly — they call
a repository method. This means: (a) business logic is testable without
FastAPI/HTTP in the loop, (b) the ORM could be swapped without touching
endpoint code, (c) authorization (`require_roles`) is declared once per
route via FastAPI's dependency system rather than re-implemented per
handler.

## Why these specific technology choices

- **Repository pattern over a service-layer-only design**: with 10+
  endpoint modules touching a handful of shared tables, a thin
  repository per aggregate (User, Booking, Revenue) was less overhead
  than a full service layer while still isolating query logic.
- **Holt-Winters + Prophet + ARIMA + XGBoost ensemble**: no single
  forecasting method is best for every property/season combination.
  Rather than commit to one, `ForecastComparisonService` backtests all
  four on held-out actuals and recommends whichever had the lowest MAPE
  — the same validate-before-trust discipline a real revenue-management
  team would apply.
- **FAISS + local sentence-transformer embeddings for RAG**, not a
  managed vector DB: keeps the chatbot's knowledge-base retrieval
  runnable with zero paid infrastructure for local development, while
  still swappable for pgvector/OpenSearch in production (see
  `docs/DEPLOYMENT.md`).
- **LLM provider factory (Groq/OpenAI/Ollama)**: the chatbot depends
  only on LangChain's `BaseChatModel` interface; the concrete provider
  is a one-line `.env` change (`LLM_PROVIDER=ollama` for a fully local,
  zero-API-key setup during development).
- **Pandas-based ETL over row-by-row Python loops**: the cleaning
  (`app/etl/data_cleaning.py`) and feature engineering
  (`app/ml/feature_engineering.py`) modules are vectorized DataFrame
  operations, matching how a real data-engineering team would process a
  multi-thousand-row CSV export, and are unit-tested as pure functions
  independent of the database.
- **Docker Compose over Kubernetes** for this project's scope: one
  `docker compose up` reproduces the entire stack (db, backend,
  frontend, Prometheus, Grafana) on a laptop or single EC2 instance,
  appropriate for a portfolio project; `docs/DEPLOYMENT.md` notes the
  scaling path if this grew into a multi-tenant SaaS product.

## Known, intentional simplifications

These are documented trade-offs, not oversights — see
`FINAL_VERIFICATION.md` for the full honest audit:

- **No Alembic migrations** — schema is created via
  `Base.metadata.create_all()`. Fine for a fresh single-environment
  deploy; would need Alembic for iterative schema changes post-launch.
- **FAISS vector store is in-memory per process**, not persisted or
  shared across replicas.
- **XGBoost/ARIMA confidence intervals are approximations** (residual
  std-dev or recursive-forecast bands), not formally calibrated
  prediction intervals.
- **Power BI is documentation-only** — SQL views exist and are real, but
  no `.pbix` report file has been built (see `powerbi/README.md`).
