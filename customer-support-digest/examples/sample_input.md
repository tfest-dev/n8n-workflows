# Sample input: support tickets

The workflow stores this equivalent payload in `Set Example Ticket Inputs` as `tickets_json`.

```json
[
  {
    "id": 1234,
    "subject": "Login fails on mobile app",
    "body": "I cannot log in from the iOS app since this morning. Two team members are blocked.",
    "status": "open",
    "priority": "high",
    "tags": ["bug", "mobile", "login"],
    "customer": "Acme Corp",
    "plan": "Enterprise"
  },
  {
    "id": 1235,
    "subject": "Feature request: custom export",
    "body": "We'd like to export reports to our internal data lake in a scheduled format.",
    "status": "open",
    "priority": "normal",
    "tags": ["feature-request", "reporting"],
    "customer": "Beta Inc",
    "plan": "Pro"
  }
]
```

Example run configuration:

```json
{
  "digest_frequency": "daily",
  "lookback_hours": 24,
  "max_tickets_for_llm": 25,
  "delivery_channel": "leadership_digest",
  "include_low_priority": true,
  "use_llm": true,
  "llm_alias": "general"
}
```
