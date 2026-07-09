# Sample output: live incident object

```json
{
  "incident_id": "inc_2025_0012",
  "service": "payments-api",
  "severity": "SEV1",
  "summary": "Payments API latency > 5s for 30% of requests",
  "environment": "production",
  "owner_team": "payments-platform",
  "on_call_engineer": "alice@yourcompany.com",
  "orchestration_status": "success",
  "slack_channel": "#inc-sev1-inc_2025_0012",
  "ticket_id": "PAY-4821",
  "status_page_id": "sp_inc_2025_0012",
  "links": {
    "alert": "https://pagerduty.com/incidents/inc_2025_0012",
    "collaboration": "https://your-slack-workspace.slack.com/archives/inc-sev1-inc_2025_0012",
    "ticket": "https://your-jira-instance/browse/PAY-4821",
    "status_page": "https://status.yourcompany.com/incidents/sp_inc_2025_0012"
  },
  "enrichment_warnings": [],
  "failed_actions": []
}
```

If one action fails, the object uses `orchestration_status: "partial_success"` and includes the failed action in `failed_actions` plus an operations alert payload.
