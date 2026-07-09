# Sample output: enriched and scored lead

```json
{
  "email": "jane.doe@acme.com",
  "name": "Jane Doe",
  "company": "Acme Corp",
  "website": "https://acme.com",
  "source": "Landing page - ebook download",
  "utm_campaign": "q1_pipeline_boost",
  "utm_medium": "paid-social",
  "enrichment": {
    "legal_name": "Acme Corp",
    "employee_count": 350,
    "employee_count_band": "201-500",
    "industry": "SaaS",
    "country": "US",
    "annual_revenue_band": "10m-50m",
    "funding_stage": "Series B",
    "tech_stack": ["AWS", "PostgreSQL", "HubSpot", "Segment"],
    "has_crm": true,
    "has_cloud_stack": true,
    "has_data_stack": true
  },
  "score": 94,
  "segment": "enterprise",
  "route": "enterprise_sales",
  "score_reasons": [
    "Mid-market employee count",
    "Industry fit: SaaS",
    "Supported sales region: US",
    "CRM or revenue tooling detected",
    "Modern cloud stack detected",
    "Recent pricing-page activity",
    "Content engagement present",
    "Existing CRM account context available"
  ],
  "score_confidence": "high",
  "should_notify_sales": true,
  "qualification_mode": "llm",
  "llm_summary": "## Account summary
Acme Corp is a strong-fit SaaS account with mid-market scale and modern revenue tooling.

## Qualification notes
- Route to enterprise or mid-market sales depending on territory ownership.
- Prioritise fast follow-up because pricing-page activity and campaign engagement are present.",
  "overall_status": "success"
}
```

The full workflow output also contains CRM upsert payloads, sales notification payloads, source status objects, operations alerts and an audit record.
