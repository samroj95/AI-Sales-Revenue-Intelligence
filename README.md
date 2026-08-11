# AI Sales & Revenue Intelligence Platform

An AI-powered hospitality revenue intelligence platform: FastAPI +
PostgreSQL backend, React/Tailwind frontend, ML forecasting (Holt-Winters,
Prophet, ARIMA, XGBoost), anomaly detection, a RAG-based AI chatbot
(Groq/OpenAI/Ollama), and threshold-based alerts.

> **Before you run this**: read `FINAL_VERIFICATION.md` for an honest,
> line-by-line audit of what is implemented, partially implemented, or
> not verified in this codebase. This README describes the intended
> design; the verification doc describes what has actually been checked.

## Quick facts

| | |
|---|---|
| Backend | Python 3.11, FastAPI, SQLAlchemy 2.0 |
| Database | PostgreSQL 16 |
| Frontend | React 18, Tailwind CSS, Vite |
| ML | scikit-learn, XGBoost, Prophet, statsmodels (ARIMA/Holt-Winters) |
| AI | LangChain + FAISS (RAG), Groq / OpenAI / Ollama (swappable) |
| Auth | JWT, 5 roles (admin, revenue_manager, general_manager, regional_manager, finance) |
| Deployment | Docker Compose (db, backend, frontend, Prometheus, Grafana) |

## Getting started

For Windows, follow **`SETUP_WINDOWS.md`** step by step. For macOS/Linux,
the short version:

```bash
cp backend/.env.example backend/.env      # then edit backend/.env with your LLM API key
docker compose up --build
```

- Backend API docs: http://localhost:8000/api/v1/docs
- Frontend: http://localhost:3000
- Default login: `admin@sales-intel.ai` / `Admin@12345` (seeded automatically on first boot, along with a synthetic demo dataset — see `backend/app/etl/generate_synthetic_data.py`)
- Grafana: http://localhost:3001 (admin/admin)
- Prometheus: http://localhost:9090

## Project structure

```
backend/            FastAPI application (see backend/app/ for modules)
  app/api/           REST endpoints
  app/models/        SQLAlchemy ORM models
  app/schemas/        Pydantic request/response schemas
  app/repositories/   Data-access layer (Repository pattern)
  app/etl/            Data ingestion, cleaning, ETL pipeline, synthetic data generator
  app/ml/             Forecasting (4 models) + anomaly detection + feature engineering
  app/ai/             RAG chatbot (LangChain + FAISS + swappable LLM provider)
  app/alerts/          Email/Slack/AI-recommendation alert engine
  sql/                50 analytics queries, views, indexes, 1 stored procedure
  tests/              pytest suite
frontend/            React + Tailwind SPA
monitoring/          Prometheus + Grafana config
powerbi/             Power BI connection guide (NOT a working .pbix — see powerbi/README.md)
.github/workflows/   CI/CD (GitHub Actions)
```

## Documentation index

- `INSTALLATION_REQUIREMENTS.md` — exact software you need on your machine
- `SETUP_WINDOWS.md` — step-by-step Windows setup
- `FINAL_VERIFICATION.md` — honest completion audit (what's tested vs. assumed)
- `docs/API.md` — API endpoint reference
- `docs/DEPLOYMENT.md` — AWS EC2 deployment guide
- `powerbi/README.md` — Power BI connection guide (documentation only)

## License

Provided as a portfolio/reference project. No warranty; review and
harden before any production use (see security notes in `docs/DEPLOYMENT.md`).
