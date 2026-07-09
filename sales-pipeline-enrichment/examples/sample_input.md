# Sample input: new lead webhook

```json
{
  "email": "jane.doe@acme.com",
  "name": "Jane Doe",
  "company": "Acme Corp",
  "website": "https://acme.com",
  "source": "Landing page - ebook download",
  "utm_campaign": "q1_pipeline_boost",
  "utm_medium": "paid-social",
  "lead_source_id": "lead_demo_001"
}
```

## Degraded-source test payload

```json
{
  "email": "jane.doe@acme.com",
  "name": "Jane Doe",
  "company": "Acme Corp",
  "website": "https://acme.com",
  "source": "Landing page - ebook download",
  "utm_campaign": "q1_pipeline_boost",
  "utm_medium": "paid-social",
  "simulate_source_status": { "company": "partial", "tech_stack": "failed" }
}
```
