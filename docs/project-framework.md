# Project Framework

## 1. Project Definition

ExpenseAnalysis is an AI-native expense analysis product with the following pipeline:

```text
User message
→ parser
→ normalized transaction
→ database
→ statistics engine
→ analysis generator
→ correction feedback
→ memory update
```

The product should optimize for **speed of entry, stability of classification, trustworthiness of numbers, and useful AI explanations**.

---

## 2. MVP Scope

### In Scope
- Single-sentence expense input.
- Automatic amount/category/merchant/time parsing.
- Transaction persistence.
- Transaction list and filters.
- Weekly/monthly deterministic stats.
- AI-generated weekly summary based on computed stats.
- User correction for wrong category or merchant.

### Out of Scope for MVP
- Multi-user collaboration.
- Bank account synchronization.
- Receipt OCR.
- Budget automation.
- Complex forecasting.
- Native mobile apps.

---

## 3. Core User Flows

### Flow A: Quick Entry
1. User types `午饭 25`.
2. Parser extracts amount `25`, category `food`, merchant `null`, date `today`.
3. User optionally confirms.
4. Transaction is saved.

### Flow B: Review and Correct
1. User opens list.
2. User sees a record categorized incorrectly.
3. User edits category.
4. System updates the transaction and stores category memory.

### Flow C: View Stats and Analysis
1. User opens weekly dashboard.
2. Backend computes weekly totals, category distribution, and comparison to previous week.
3. LLM generates a narrative summary using computed stats.

---

## 4. System Modules

### 4.1 Parsing Service
Responsibilities:
- extract money amounts with rules/regex
- classify category using fixed enum
- identify merchant keywords
- infer transaction date/time
- assign confidence score
- return structured JSON that passes schema validation

Implementation principles:
- rules first for obvious values
- LLM for semantic understanding
- schema validation before persistence
- unresolved ambiguity should return review flags

### 4.2 Transaction Service
Responsibilities:
- create transaction
- update transaction
- list transactions
- soft delete or archive transaction if needed
- attach parsing metadata and raw input

### 4.3 Memory Service
Responsibilities:
- remember user category corrections
- keep merchant → category mappings
- boost parser confidence using learned mappings

### 4.4 Stats Service
Responsibilities:
- compute totals by time range
- compute category ratios
- compute week-over-week or month-over-month deltas
- detect unusual spending

Important rule:
- all arithmetic must be deterministic and code-based

### 4.5 Analysis Service
Responsibilities:
- convert computed stats into human-readable explanations
- generate concise insights and suggestions
- never invent numbers that are not present in the stats payload

### 4.6 Frontend Experience
Responsibilities:
- ultra-simple quick input
- clear parse preview
- transaction list with filters
- stats visualization
- analysis text block
- lightweight correction loop

---

## 5. Category System

Use a **fixed category enum** to keep analytics stable.

Recommended MVP categories:
- food
- transport
- shopping
- groceries
- entertainment
- housing
- utilities
- healthcare
- education
- travel
- subscription
- transfer
- other

Rules:
- all transactions must map to one enum value
- parser cannot create new categories dynamically
- user corrections update memory, not the enum list

---

## 6. Data Model

### 6.1 transactions
| Field | Type | Description |
|---|---|---|
| id | UUID/int | primary key |
| raw_text | string | original user input |
| amount | decimal | normalized amount |
| currency | string | default `CNY` or configurable |
| category | enum | normalized category |
| merchant | string nullable | parsed merchant |
| transaction_date | datetime/date | inferred date |
| note | string nullable | optional extra note |
| parse_confidence | float | parser confidence 0-1 |
| parse_source | string | `mock`, `rules`, `llm`, `hybrid` |
| needs_review | bool | whether user should confirm |
| created_at | datetime | record created time |
| updated_at | datetime | last update time |

### 6.2 category_memory
| Field | Type | Description |
|---|---|---|
| id | UUID/int | primary key |
| merchant_key | string | normalized merchant token |
| raw_pattern | string nullable | original phrase or keyword |
| preferred_category | enum | learned category |
| confidence | float | memory confidence |
| source | string | `user_correction` or `seed_rule` |
| created_at | datetime | created time |
| updated_at | datetime | updated time |

### 6.3 analysis_cache
| Field | Type | Description |
|---|---|---|
| id | UUID/int | primary key |
| period_type | string | `weekly` or `monthly` |
| period_start | date | range start |
| period_end | date | range end |
| stats_payload | json | deterministic stats snapshot |
| analysis_text | text | generated summary |
| model_name | string | model used |
| created_at | datetime | created time |

---

## 7. Parsing Strategy

### Input
Free-form short messages such as:
- `午饭 25`
- `滴滴 42`
- `昨天星巴克 36`
- `买书花了68`

### Output schema
```json
{
  "raw_text": "昨天星巴克 36",
  "amount": 36,
  "currency": "CNY",
  "category": "food",
  "merchant": "星巴克",
  "transaction_date": "2026-03-22",
  "note": null,
  "parse_confidence": 0.92,
  "needs_review": false,
  "reason": "merchant match + amount extracted"
}
```

### Parsing pipeline
1. preprocess text
2. extract amount by regex/rules
3. match memory rules
4. call LLM when semantic classification is needed
5. validate against schema and enum
6. mark low-confidence items for review

### Low-confidence fallback
If parser cannot confidently classify the input:
- keep amount if reliable
- set category to `other` or null during preview stage
- set `needs_review = true`
- return a short explanation for the UI

---

## 8. Analytics Strategy

### Deterministic stats examples
- total spend in current week
- category breakdown percentage
- biggest transaction
- average daily spend
- delta vs previous week
- unusual merchant or amount spike

### LLM analysis prompt contract
Input to the model should be **computed stats**, not raw transaction math tasks.
The model should output:
- summary
- notable changes
- possible risks
- practical suggestions

Guardrails:
- do not allow the model to invent totals
- include exact values from stats payload
- require concise, user-friendly language

---

## 9. Team Split

### Backend owner
- API design
- DB schema
- parser implementation
- stats implementation
- analysis orchestration
- tests and validation

### Frontend owner
- quick input UX
- transaction list page
- stats cards/charts
- analysis page
- correction interaction

### Shared alignment items
- category enum
- API contracts
- response shapes
- error codes

---

## 10. Engineering Constraints

- parsing output must use strict schema validation
- category values must come from a fixed enum
- numeric stats must be computed in code
- all user-visible AI summaries must be traceable to a stats payload
- corrections must be persisted so the system improves over time

---

## 11. Non-Functional Requirements

- fast input workflow: create transaction in a few seconds
- reliable classification consistency
- transparent low-confidence handling
- simple deployment for demo use
- easy local development for a 2-person team

---

## 12. Phase Plan

### Phase 1: Closed-loop MVP
- mock parsing
- transaction CRUD
- weekly stats
- generated weekly summary

### Phase 2: Reliable parsing
- LLM + rule hybrid parser
- correction memory
- confidence thresholds

### Phase 3: Better product experience
- charts
- trend explanations
- anomaly detection
- richer natural language queries
