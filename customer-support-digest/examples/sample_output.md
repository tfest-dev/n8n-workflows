# Sample output: daily support digest

```markdown
# Customer Support Digest - 2026-07-08

## Summary
- 7 usable tickets included from 8 raw tickets.
- 3 high-severity tickets identified by deterministic rules.
- Leading theme: authentication.
- Mobile login and SSO issues need product/engineering review.

## Top themes
1. **authentication** (3 tickets)
2. **feature_request** (2 tickets)
3. **billing** (1 ticket)
4. **onboarding** (1 ticket)

## High-severity items
- **#1234 Acme Corp — Login fails on mobile app**
  - Suggested next step: review owner, impact and customer communication plan.
- **#1236 Globex — SSO timeout during onboarding**
  - Suggested next step: review owner, impact and customer communication plan.

## Product / engineering signals
- #1234 authentication: Login fails on mobile app
- #1235 feature_request: Feature request: custom export
- #1239 authentication: Mobile login still failing after password reset

## Ticket list
- #1234 Acme Corp — Login fails on mobile app (high/high)
- #1235 Beta Inc — Feature request: custom export (normal/normal)
```

The final audit output also includes:

```json
{
  "audit_type": "customer_support_digest",
  "digest_generation_mode": "llm",
  "source_statuses": {
    "helpdesk": {
      "status": "success",
      "records_returned": 8
    }
  },
  "ticket_summary": {
    "raw_count": 8,
    "included_count": 7,
    "dropped_count": 1,
    "high_severity_count": 3
  },
  "leadership_notification": {
    "status": "prepared_replace_with_email_slack_or_teams_node"
  }
}
```
