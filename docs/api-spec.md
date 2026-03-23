# API Specification

Base path suggestion: `/api/v1`

## Common Conventions

### Headers
- `Content-Type: application/json`

### Response envelope (recommended)
```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {}
}
```

### Error object
```json
{
  "code": "VALIDATION_ERROR",
  "message": "category is invalid",
  "details": {}
}
```

### Category enum
```json
[
  "food",
  "transport",
  "shopping",
  "groceries",
  "entertainment",
  "housing",
  "utilities",
  "healthcare",
  "education",
  "travel",
  "subscription",
  "transfer",
  "other"
]
```

---

## 1. Health Check

### `GET /health`
Purpose: verify service availability.

#### Response
```json
{
  "success": true,
  "data": {
    "status": "ok"
  },
  "error": null,
  "meta": {}
}
```

---

## 2. Parse Transaction Preview

### `POST /transactions/parse`
Purpose: parse a natural-language input into a structured preview before saving.

#### Request
```json
{
  "text": "昨天星巴克 36",
  "timezone": "Asia/Shanghai",
  "default_currency": "CNY"
}
```

#### Response
```json
{
  "success": true,
  "data": {
    "raw_text": "昨天星巴克 36",
    "amount": 36,
    "currency": "CNY",
    "category": "food",
    "merchant": "星巴克",
    "transaction_date": "2026-03-22",
    "note": null,
    "parse_confidence": 0.92,
    "parse_source": "hybrid",
    "needs_review": false,
    "reason": "amount extracted and merchant matched"
  },
  "error": null,
  "meta": {}
}
```

#### Validation rules
- `text` is required.
- `amount` must be positive.
- `category` must be in enum.
- `transaction_date` should be returned in ISO format.

---

## 3. Create Transaction

### `POST /transactions`
Purpose: persist a parsed transaction.

#### Request
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
  "parse_source": "hybrid",
  "needs_review": false
}
```

#### Response
```json
{
  "success": true,
  "data": {
    "id": 101,
    "raw_text": "昨天星巴克 36",
    "amount": 36,
    "currency": "CNY",
    "category": "food",
    "merchant": "星巴克",
    "transaction_date": "2026-03-22",
    "note": null,
    "parse_confidence": 0.92,
    "parse_source": "hybrid",
    "needs_review": false,
    "created_at": "2026-03-23T10:00:00Z",
    "updated_at": "2026-03-23T10:00:00Z"
  },
  "error": null,
  "meta": {}
}
```

---

## 4. List Transactions

### `GET /transactions`
Purpose: query transaction history.

#### Query parameters
- `start_date` (optional, ISO date)
- `end_date` (optional, ISO date)
- `category` (optional)
- `merchant` (optional)
- `page` (optional, default `1`)
- `page_size` (optional, default `20`)
- `needs_review` (optional, boolean)

#### Example
`GET /transactions?start_date=2026-03-01&end_date=2026-03-31&category=food&page=1&page_size=20`

#### Response
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 101,
        "raw_text": "昨天星巴克 36",
        "amount": 36,
        "currency": "CNY",
        "category": "food",
        "merchant": "星巴克",
        "transaction_date": "2026-03-22",
        "note": null,
        "parse_confidence": 0.92,
        "parse_source": "hybrid",
        "needs_review": false,
        "created_at": "2026-03-23T10:00:00Z",
        "updated_at": "2026-03-23T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total": 1
    }
  },
  "error": null,
  "meta": {}
}
```

---

## 5. Get Transaction Detail

### `GET /transactions/{transaction_id}`
Purpose: fetch one transaction by id.

---

## 6. Update Transaction / Correct Parsing

### `PATCH /transactions/{transaction_id}`
Purpose: allow user corrections and persist improved values.

#### Request
```json
{
  "category": "groceries",
  "merchant": "盒马",
  "note": "user corrected category"
}
```

#### Response
```json
{
  "success": true,
  "data": {
    "id": 101,
    "category": "groceries",
    "merchant": "盒马",
    "updated_at": "2026-03-23T10:10:00Z"
  },
  "error": null,
  "meta": {}
}
```

### Side effects
- if correction changes category, write or update `category_memory`
- clear `needs_review` if user confirms final values

---

## 7. Delete Transaction

### `DELETE /transactions/{transaction_id}`
Purpose: remove a transaction or mark it deleted.

#### Response
```json
{
  "success": true,
  "data": {
    "deleted": true
  },
  "error": null,
  "meta": {}
}
```

---

## 8. Monthly Stats

### `GET /stats/monthly`
Purpose: return deterministic monthly statistics.

#### Query parameters
- `year` required
- `month` required
- `timezone` optional

#### Response
```json
{
  "success": true,
  "data": {
    "period": {
      "type": "monthly",
      "start": "2026-03-01",
      "end": "2026-03-31"
    },
    "total_spend": 520,
    "transaction_count": 18,
    "average_daily_spend": 16.77,
    "top_category": "food",
    "category_breakdown": [
      {"category": "food", "amount": 234, "ratio": 0.45},
      {"category": "transport", "amount": 88, "ratio": 0.17}
    ],
    "largest_transaction": {
      "id": 88,
      "amount": 120,
      "category": "shopping",
      "merchant": "优衣库",
      "transaction_date": "2026-03-12"
    },
    "comparison": {
      "previous_period_total": 430,
      "delta_amount": 90,
      "delta_ratio": 0.2093
    }
  },
  "error": null,
  "meta": {}
}
```

---

## 9. Weekly Analysis

### `GET /analysis/weekly`
Purpose: return computed weekly stats plus an LLM-generated explanation.

#### Query parameters
- `week_start` required (ISO date)
- `timezone` optional
- `refresh` optional boolean to bypass cache

#### Response
```json
{
  "success": true,
  "data": {
    "period": {
      "type": "weekly",
      "start": "2026-03-16",
      "end": "2026-03-22"
    },
    "stats": {
      "total_spend": 520,
      "transaction_count": 18,
      "top_category": "food",
      "category_breakdown": [
        {"category": "food", "amount": 234, "ratio": 0.45},
        {"category": "transport", "amount": 88, "ratio": 0.17}
      ],
      "comparison": {
        "previous_period_total": 430,
        "delta_amount": 90,
        "delta_ratio": 0.2093
      },
      "anomalies": [
        {
          "type": "large_transaction",
          "message": "One shopping transaction reached 120, above your recent average."
        }
      ]
    },
    "analysis": {
      "summary": "This week you spent 520 in total. Food remained the largest category at 45% of spending.",
      "insights": [
        "Weekly spending increased by 20.9% versus the previous week.",
        "Transport remained moderate, but food spending dominated overall behavior."
      ],
      "suggestions": [
        "Consider setting a soft cap for dining out next week.",
        "Review whether high-frequency small purchases are avoidable."
      ],
      "generated_at": "2026-03-23T10:15:00Z",
      "model_name": "gpt-4.1-mini"
    }
  },
  "error": null,
  "meta": {}
}
```

---

## 10. Category Memory List

### `GET /category-memory`
Purpose: inspect learned merchant/category mappings.

#### Response fields
- `merchant_key`
- `preferred_category`
- `confidence`
- `source`

---

## 11. Upsert Category Memory

### `POST /category-memory`
Purpose: manually seed or update category memory.

#### Request
```json
{
  "merchant_key": "星巴克",
  "preferred_category": "food",
  "confidence": 0.95,
  "source": "user_correction"
}
```

---

## 12. Rebuild Weekly Analysis Cache

### `POST /analysis/weekly/rebuild`
Purpose: regenerate weekly cached analysis after data corrections.

#### Request
```json
{
  "week_start": "2026-03-16",
  "timezone": "Asia/Shanghai"
}
```

---

## 13. Suggested Backend Schema Objects

### ParseTransactionRequest
```json
{
  "text": "string",
  "timezone": "string",
  "default_currency": "string"
}
```

### TransactionCreateRequest
```json
{
  "raw_text": "string",
  "amount": 0,
  "currency": "string",
  "category": "food",
  "merchant": "string",
  "transaction_date": "2026-03-22",
  "note": "string",
  "parse_confidence": 0,
  "parse_source": "hybrid",
  "needs_review": false
}
```

### TransactionUpdateRequest
```json
{
  "category": "food",
  "merchant": "string",
  "transaction_date": "2026-03-22",
  "note": "string",
  "needs_review": false
}
```
