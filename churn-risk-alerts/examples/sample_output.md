# Sample output

The final node returns a payload containing the Markdown watchlist, operations alert payloads, prepared outreach tasks and an audit record.

```markdown
# Churn Risk Watchlist

Run date: 2026-07-08
Scoring version: churn-risk-rules-v2
Threshold: 65
Accounts on watchlist: 2
Aggregation status: success

## Source status
- usage: success | 4 records | checked 2026-07-08T09:00:00.000Z
- billing: success | 4 records | checked 2026-07-08T09:00:00.000Z
- crm: success | 4 records | checked 2026-07-08T09:00:00.000Z
- support: success | 4 records | checked 2026-07-08T09:00:00.000Z

## Priority accounts
- **Northstar Logistics** (acct_northstar) — critical risk, score 100, MRR 2800
  - Owner: Daniel Green
  - Renewal: 2026-07-28 (20 days)
  - Reasons: No product activity for 36 days; Usage down 78%; Low seat utilisation at 16%; Renewal in 20 days; Recent failed payment
  - Outreach: Suggested outreach: Daniel Green should contact Northstar Logistics with a practical recovery plan...
```

Example outreach task payload:

```json
{
  "task_type": "customer_success_outreach",
  "account_id": "acct_northstar",
  "account_name": "Northstar Logistics",
  "owner": "Daniel Green",
  "priority": "critical",
  "risk_score": 100,
  "idempotency_key": "cs-outreach:acct_northstar:2026-07-28:churn-risk-rules-v2",
  "title": "Churn-risk follow-up: Northstar Logistics",
  "due_date_hint": "within_2_business_days"
}
```

If a source fails, the output also includes `operations_alerts` and `source_warnings` so customer success can see which signals were missing from the score.
