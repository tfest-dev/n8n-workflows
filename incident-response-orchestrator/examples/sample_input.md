# Sample input: SEV1 alert payload

```json
{
  "incident_id": "inc_2025_0012",
  "service": "payments-api",
  "severity": "SEV1",
  "summary": "Payments API latency > 5s for 30% of requests",
  "url": "https://pagerduty.com/incidents/inc_2025_0012",
  "environment": "production",
  "simulate_failures": {}
}
```

To test degraded orchestration, use:

```json
{
  "incident_id": "inc_2025_0012",
  "service": "payments-api",
  "severity": "SEV1",
  "summary": "Payments API latency > 5s for 30% of requests",
  "url": "https://pagerduty.com/incidents/inc_2025_0012",
  "environment": "production",
  "simulate_failures": {
    "status_page": true
  }
}
```
