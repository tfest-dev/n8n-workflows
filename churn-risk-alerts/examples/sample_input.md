# Sample input

The workflow uses separate source payloads for usage, billing, CRM and support signals.

```json
{
  "risk_threshold": 65,
  "top_n": 10,
  "lookback_days": 30,
  "simulate_source_failures": {},
  "usage_signals": [
    {
      "account_id": "acct_northstar",
      "last_active_date": "2026-06-02",
      "logins_30d": 1,
      "feature_events_30d": 3,
      "seats_used": 4,
      "seats_purchased": 25,
      "usage_trend_pct": -78
    }
  ],
  "billing_signals": [
    {
      "account_id": "acct_northstar",
      "plan": "Growth",
      "mrr": 2800,
      "renewal_date": "2026-07-28",
      "payment_status": "failed",
      "downgrade_requested": true
    }
  ],
  "crm_signals": [
    {
      "account_id": "acct_northstar",
      "account_name": "Northstar Logistics",
      "owner": "Daniel Green",
      "health_status": "red",
      "executive_sponsor_engaged": false
    }
  ],
  "support_signals": [
    {
      "account_id": "acct_northstar",
      "high_priority_tickets_30d": 5,
      "open_tickets": 9,
      "csat_avg": 2.1,
      "nps_score": 1,
      "themes": ["billing issue", "poor adoption", "renewal uncertainty"]
    }
  ]
}
```

To test source failure handling, set:

```json
{"support": true}
```

in `simulate_source_failures`.
