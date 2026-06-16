# Operational Monitoring & Responsible-Use Architecture

## 1. Post-Deployment Monitoring Targets

To keep this churn engine running reliably after deployment, the engineering team must track four core performance domains:

### A. Feature & Data Drift Metrics
* **What to Track:** Population Stability Index (PSI) and Wasserstein Distance across numerical features (such as `recency_days` and `last_visit_days_ago`).
* **Trigger Condition:** If the incoming 7-day rolling feature distributions deviate from the baseline dataset splits with a $PSI > 0.2$, trigger an automated warning. This likely means user behavior has shifted significantly (e.g., due to a major holiday app update).

### B. Live Prediction Distribution Tracking
* **What to Track:** The daily ratio of predicted churners versus standard non-churners.
* **Trigger Condition:** Our baseline model assumes a churn rate of roughly **46.9%**. If the live production inference stream sudden drops below 25% or spikes past 65% for more than 48 consecutive hours, pause automated campaign flows. This indicates an upstream logging bug or pipeline failure.

### C. Downstream Business Outcomes
* **What to Track:** The actual customer conversion rate and return-to-platform metrics following retention coupon distribution.
* **Trigger Condition:** If the control group (no discount) and the treatment group (received discount coupon) show identical 60-day survival metrics, the retention strategies are causing margin erosion without changing customer behavior. This requires an immediate rewrite of marketing campaign rules.

### D. System Performance & Infrastructure Metrics
* **What to Track:** API Request-Response latency profiles (p95 and p99 intervals) along with HTTP status code distributions (`500 Internal Server Errors` vs. `422 Unprocessable Entities`).
* **Trigger Condition:** Latency must remain below **50ms** per record. Any spike in `500 errors` exceeding 1% of total daily traffic will automatically trigger a developer alert.

---

## 2. Model Retraining Schedule
* **Cadence Trigger:** The churn engine will be retrained **monthly**, using a rolling window of active user transaction history.
* **Emergency Trigger:** If validation accuracy metrics drop below **78%** or the $F1-Score$ drops below **0.75** during weekly validation checks against fresh, labeled history logs, an immediate manual model audit and retraining loop will be initiated.

---

## 3. Ethical Safeguards & Responsible-Use Guidelines

### Approved Use Cases ✅
* Deploying proactive customer service calls to clients in the "Frustrated / High-Complaint" category.
* Distributing educational product onboarding emails or offering light, high-margin product bundle discounts to re-engage "At-Risk" users.

### Explicitly Prohibited Use Cases ❌
* **Dynamic Price Gouging:** Increasing checkout prices for users with low churn risks due to their high brand dependency.
* **Service Level Blacklisting:** De-prioritizing delivery speeds or completely blocking support channels for users flagged with high churn probabilities. The model must only be used as a positive mechanism for retention, never as a tool to penalize users.
