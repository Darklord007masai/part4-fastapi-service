# Part 4 — FastAPI Churn Scoring Service

D2C Customer Churn Capstone. Wraps the Part 3 model in a simple internal API the
retention/CRM team can call.

## Endpoints
- `GET /health` — service + model status
- `POST /predict` — score one customer
- `POST /batch_predict` — score many customers

## Setup & run
```bash
pip install -r requirements.txt

# 1) build the model (needs the modelling CSV from Part 3)
python train_model.py --data rfm_modeling_snapshot.csv   # writes model.pkl

# 2) start the API
uvicorn app.main:app --reload
# open http://127.0.0.1:8000/docs for interactive Swagger UI
```

> Note: this part runs on a real Python environment, not inside a Colab cell.
> Easiest options: run locally as above, or deploy free on Render
> (Build: `pip install -r requirements.txt && python train_model.py --data rfm_modeling_snapshot.csv`,
> Start: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`).

## Sample request (`POST /predict`)
```json
{
  "city_tier": "Tier 1", "age_group": "25-34", "acquisition_channel": "Instagram",
  "loyalty_tier": null, "preferred_category": "Skin Care", "marketing_consent": "Yes",
  "recency_days": 200, "frequency_180d": 1, "monetary_180d": 450.0,
  "return_rate_180d": 0.0, "avg_discount_pct_180d": 0.1, "avg_rating_180d": 4.0,
  "category_diversity_180d": 1, "ticket_count_90d": 0, "negative_ticket_rate_90d": 0.0,
  "avg_resolution_hours_90d": 0.0, "days_since_signup": 300, "sessions_30d": 0,
  "product_views_30d": 0, "cart_adds_30d": 0, "wishlist_adds_30d": 0,
  "abandoned_carts_30d": 0, "email_opens_30d": 0, "campaign_clicks_30d": 0,
  "last_visit_days_ago": 60
}
```

## Sample response
```json
{
  "churn_probability": 0.8918,
  "churn_prediction": 1,
  "risk_band": "High",
  "risk_explanation": "no purchase in 120+ days; not active on site in 30+ days; low order frequency"
}
```

## Tests
```bash
pytest -q     # 4 tests: health, predict, batch_predict, invalid-input rejection
```

## Files
- `app/main.py` — FastAPI app
- `app/schemas.py` — Pydantic request/response validation
- `train_model.py` — trains and saves `model.pkl`
- `tests/test_api.py` — API tests
- `Dockerfile` — optional containerised run
- `monitoring_plan.md` — post-deployment monitoring + responsible use

## Docker (optional)
```bash
docker build -t churn-api .
docker run -p 8000:8000 -v $(pwd)/model.pkl:/code/model.pkl churn-api
```
