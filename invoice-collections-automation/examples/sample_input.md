# Sample input: overdue invoices

```json
[
  {
    "invoice_id": "inv_1001",
    "customer_name": "Acme Corp",
    "customer_email": "billing@acme.com",
    "owner_email": "jane@yourcompany.com",
    "amount_due": 4200,
    "currency": "USD",
    "days_overdue": 12,
    "status": "past_due",
    "written_off": false,
    "last_reminder_sent_at": null,
    "invoice_url": "https://billing.example.test/invoices/inv_1001",
    "account_id": "acct_acme"
  },
  {
    "invoice_id": "inv_1002",
    "customer_name": "Beta Inc",
    "customer_email": "ap@beta.inc",
    "owner_email": "tom@yourcompany.com",
    "amount_due": 9800,
    "currency": "USD",
    "days_overdue": 45,
    "status": "past_due",
    "written_off": false,
    "last_reminder_sent_at": "2026-07-01",
    "invoice_url": "https://billing.example.test/invoices/inv_1002",
    "account_id": "acct_beta"
  }
]
```
