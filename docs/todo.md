# TODO

## 0. Product Alignment
- [ ] Freeze MVP scope and explicitly reject non-MVP features.
- [ ] Freeze category enum for backend and frontend shared usage.
- [ ] Confirm single-user assumptions for MVP.
- [ ] Decide default currency, locale, and timezone behavior.

## 1. Repository Bootstrap
- [ ] Initialize `backend/` with FastAPI app skeleton.
- [ ] Initialize `frontend/` with React + Vite app skeleton.
- [ ] Add shared `.editorconfig`, `.gitignore`, and formatting tools.
- [ ] Add environment variable examples for backend/frontend.

## 2. Backend Foundation
- [ ] Create FastAPI app entrypoint and router registration.
- [ ] Configure SQLAlchemy engine and session management.
- [ ] Define Pydantic request/response schemas.
- [ ] Add category enum and shared constants.
- [ ] Add database migration strategy (Alembic or simple bootstrap for MVP).

## 3. Database Models
- [ ] Implement `transactions` model.
- [ ] Implement `category_memory` model.
- [ ] Implement `analysis_cache` model.
- [ ] Add indexes for date, category, merchant, and period queries.

## 4. Parsing Service
- [ ] Build amount extraction with regex/rules.
- [ ] Build date phrase parsing (`today`, `yesterday`, `昨天`, etc.).
- [ ] Build merchant normalization rules.
- [ ] Implement LLM prompt for category classification.
- [ ] Add strict JSON schema validation for model output.
- [ ] Add confidence score logic and review threshold.
- [ ] Add mock parser mode for development/demo.

## 5. Transaction APIs
- [ ] Implement `POST /transactions/parse`.
- [ ] Implement `POST /transactions`.
- [ ] Implement `GET /transactions`.
- [ ] Implement `GET /transactions/{id}`.
- [ ] Implement `PATCH /transactions/{id}`.
- [ ] Implement `DELETE /transactions/{id}`.

## 6. Memory and Learning
- [ ] Auto-write merchant/category memory on user correction.
- [ ] Implement `GET /category-memory`.
- [ ] Implement `POST /category-memory`.
- [ ] Apply memory matches before LLM classification.
- [ ] Add tests for repeated merchant classification stability.

## 7. Stats Service
- [ ] Implement monthly aggregation query.
- [ ] Implement weekly aggregation query.
- [ ] Compute category ratios and deltas.
- [ ] Compute largest transaction and average daily spend.
- [ ] Add anomaly detection heuristics.
- [ ] Implement `GET /stats/monthly`.

## 8. Analysis Service
- [ ] Define analysis prompt using deterministic stats payload only.
- [ ] Implement weekly analysis cache strategy.
- [ ] Implement `GET /analysis/weekly`.
- [ ] Implement `POST /analysis/weekly/rebuild`.
- [ ] Ensure generated text never fabricates missing values.

## 9. Frontend Pages
- [ ] Build quick input page with parse preview.
- [ ] Build transaction list page with filtering.
- [ ] Build transaction correction form.
- [ ] Build monthly stats dashboard.
- [ ] Build weekly analysis panel.
- [ ] Add empty/loading/error states.

## 10. Frontend API Layer
- [ ] Add typed API client for all endpoints.
- [ ] Add shared transaction/category types.
- [ ] Add optimistic refresh or query invalidation strategy.

## 11. Quality and Testing
- [ ] Add parser unit tests for common Chinese and English phrases.
- [ ] Add API tests for all endpoints.
- [ ] Add stats correctness tests.
- [ ] Add regression tests for user correction memory.
- [ ] Add manual end-to-end demo script.

## 12. Demo Readiness
- [ ] Seed example transactions for presentation.
- [ ] Prepare one-click local startup instructions.
- [ ] Add screenshots/GIFs after UI exists.
- [ ] Prepare a short demo story: input → correction → stats → AI summary.

## 13. Nice-to-Have After MVP
- [ ] Natural-language query like `这周吃饭花了多少？`.
- [ ] Budget threshold reminders.
- [ ] Merchant alias management UI.
- [ ] Export CSV.
- [ ] OCR receipt ingestion.
