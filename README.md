# ExpenseAnalysis

ExpenseAnalysis is an **LLM-native personal expense analysis system**.
It is not just a bookkeeping app; it is an AI agent workflow that turns natural-language spending input into structured records, deterministic statistics, and explainable insights.

## Product Vision

```text
Natural language input
→ structured parsing
→ persistent storage
→ deterministic statistics
→ AI-generated explanation
→ user feedback learning
```

## MVP Goal

Build the smallest closed loop that supports:

1. User enters a sentence like `午饭 25` or `滴滴 42`.
2. System extracts amount, category, merchant, and time.
3. Parsed transaction is stored in the database.
4. User can view transaction history.
5. User can view stats and an AI summary.

## Core Principles

- **LLM does understanding, not arithmetic.**
- **Statistics must be computed by program logic.**
- **Categories must stay stable over time.**
- **Low-confidence parsing must be reviewable.**
- **User corrections become memory.**

## Suggested Tech Stack

### Backend
- FastAPI
- SQLAlchemy
- SQLite for MVP
- Pydantic
- OpenAI-compatible LLM integration

### Frontend
- React
- Vite
- Axios
- Minimal charting library if needed

## High-Level Architecture

```text
Frontend UI
  ├─ Quick input page
  ├─ Transaction list
  ├─ Stats dashboard
  └─ Analysis panel

Backend API
  ├─ Parsing Service      (rules + LLM + schema validation)
  ├─ Transaction Service  (CRUD + correction flow)
  ├─ Stats Service        (deterministic aggregation)
  ├─ Analysis Service     (LLM explanation over computed stats)
  └─ Memory Service       (user category preferences)

Storage
  ├─ transactions
  ├─ category_memory
  └─ analysis_cache
```

## Recommended Repository Layout

```text
ExpenseAnalysis/
├─ README.md
├─ docs/
│  ├─ project-framework.md
│  ├─ api-spec.md
│  └─ todo.md
├─ backend/
│  ├─ app/
│  │  ├─ main.py
│  │  ├─ api/
│  │  ├─ models/
│  │  ├─ schemas/
│  │  ├─ services/
│  │  ├─ repositories/
│  │  ├─ prompts/
│  │  └─ core/
│  └─ tests/
└─ frontend/
   ├─ src/
   │  ├─ pages/
   │  ├─ components/
   │  ├─ services/
   │  ├─ hooks/
   │  └─ types/
   └─ public/
```

## Delivery Roadmap

### Day 0: Define the contract
- Finalize category enum.
- Finalize database schema.
- Finalize API request/response contracts.
- Align frontend/backend ownership.

### Day 1: Mock end-to-end flow
- Add mock parser endpoint.
- Add transaction create/list endpoints.
- Build frontend input and list pages.
- Ensure input → parsed result → saved record → list display works.

### Day 2: Replace mocks with real logic
- Integrate LLM parser with strict JSON schema.
- Add deterministic stats service.
- Add weekly analysis endpoint.
- Add correction flow and category memory.

### Day 3: Demo-ready polish
- Improve error states.
- Improve confidence handling.
- Add charts and analysis panel.
- Polish UX and empty states.

## Documentation Index

- Project framework: `docs/project-framework.md`
- API specification: `docs/api-spec.md`
- Implementation TODOs: `docs/todo.md`
