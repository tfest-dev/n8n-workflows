# Sales Pipeline Enrichment

This workflow enriches new leads or deals, scores them, and prepares routing actions for the correct sales motion.

The flow is built for the common gap between lead capture and useful sales action: a form submission arrives with limited context, sales reps then lose time researching the company, checking CRM history, deciding whether it is worth immediate follow-up, and choosing the right channel.

**Workflow completed with no LLM, using the fallback**

<img src=examples/images/flow-executed-n8n-no-llm-path.png alt="Successful execution in n8n"/>

**Workflow completed with LLM use**

<img src=examples/images/flow-executed-n8n-llm-path.png alt="Successful execution in n8n"/>

## Business value

- Reduces manual research and data entry for sales representatives.
- Adds firmographic, technology, behavioural-intent and CRM-history context to new leads.
- Prioritises leads with transparent scoring rules rather than opaque qualification notes.
- Routes high-fit leads to the correct sales team quickly.
- Creates CRM, notification and logging payloads with idempotency keys so retries do not create duplicate records.
- Captures degraded enrichment sources and action failures for operational follow-up.

## How the flow works

1. `Manual Trigger (dev)` supports local testing.
2. `Lead Webhook (prod placeholder)` provides the production-shaped entry point for website forms, landing pages, ad forms or CRM webhooks.
3. `Set Example Lead Input` creates a sample lead for local execution.
4. `Validate Lead / Control Config` validates required fields, normalises the email and domain, applies default thresholds, records LLM policy and creates idempotency keys.
5. Enrichment runs in separate branches:
   - `Enrich Company Data`
   - `Enrich Tech Stack`
   - `Enrich Intent Signals`
   - `Lookup CRM History`
6. Merge nodes combine the enrichment branches using position-based merge configuration compatible with n8n 2.8.4.
7. `Aggregate Enrichment Results` builds one enriched lead object and records source warnings for failed, partial or empty sources.
8. `Prepare Source Failure Alerts` creates operations alert payloads for degraded enrichment sources.
9. `Score and Route Lead` applies deterministic sales-fit scoring and selects the route: `enterprise_sales`, `mid_market_sales` or `self_serve_nurture`.
10. `Prepare Optional LLM Qualification Prompt` creates a data-minimised prompt for account wording only.
11. `Call Local LLM Qualification` calls `http://localhost:5001/completions` using the `prompt` request parameter.
12. `Extract Qualification Summary` normalises common LLM response shapes into `llm_summary`.
13. `Build Deterministic Qualification Fallback` creates a fallback summary if the LLM call fails or returns an unexpected response.
14. Action preparation runs in separate branches:
   - `Prepare CRM Upsert (placeholder)`
   - `Prepare Sales Notification (placeholder)`
   - `Prepare Enrichment Log (placeholder)`
15. `Build Final Enriched Lead` creates the final enriched/scored lead object and an audit record.

## Client-specific implementation requirements

| Area | Client-specific replacement |
| --- | --- |
| Lead trigger | Configure webhook authentication, payload mapping, CRM polling, ad-form trigger or product sign-up trigger. |
| Company enrichment | Replace placeholder Code node with Clearbit, Apollo, ZoomInfo, Companies House, internal data warehouse or CRM enrichment. |
| Tech-stack enrichment | Replace placeholder Code node with BuiltWith, Wappalyzer, internal telemetry or product usage data. |
| Intent signals | Replace placeholder Code node with website analytics, product analytics, ad engagement, email engagement or demo-request data. |
| CRM history | Replace placeholder Code node with HubSpot, Salesforce, Pipedrive, Zoho or internal CRM lookup. |
| CRM upsert | Replace prepared payload with native CRM connector or authenticated HTTP Request node. |
| Sales notification | Replace prepared payload with Slack, Teams, email or sales engagement platform delivery. |
| Enrichment log | Replace prepared payload with database, warehouse, spreadsheet, CRM activity or audit table write. |
| LLM endpoint | Confirm local/private endpoint, request body, authentication, timeout, retry policy and retention rules. |

## Scoring and routing model

The scoring node uses transparent rules covering:

   - employee-count band
   - industry fit
   - supported sales region
   - detected CRM/revenue tooling
   - cloud/data-stack signals
   - pricing-page or campaign intent
   - CRM account history
   - free-email-domain penalty
   - confidence reduction when enrichment sources fail

The output includes:

   - `score`
   - `segment`
   - `route`
   - `score_reasons`
   - `score_confidence`
   - `should_notify_sales`
   - `scoring_version`

The score and route are deterministic and should remain the source of truth for downstream CRM and notification decisions.

## Optional LLM usage

The LLM step is used for qualification wording only. It receives a data-minimised account-level payload after deterministic enrichment and scoring have already completed.

Recommended rules:

   - Send only account-level fields needed for the sales note.
   - Avoid unnecessary personal data beyond the business contact already present in the lead record.
   - Do not send raw browsing sessions, private CRM notes or email contents without a clear data-handling agreement.
   - Use a local or approved private endpoint when confidentiality matters.
   - Keep the deterministic score and route as the decision authority.
   - Treat LLM output as a sales note, not a qualification decision.

The included HTTP node calls:

```text
POST http://localhost:5001/completions
```

with a bounded JSON body:

```json
{
  "prompt": "...",
  "alias": "general",
  "n_predict": 120,
  "max_tokens": 120,
  "temperature": 0.1,
  "top_p": 0.85
}
```

### Local LLM response control

The local LLM call uses `localhost:5001/completions` with a `prompt` request body field and bounded generation settings (`n_predict` / `max_tokens`). The prompt asks for a short Markdown note only. The extractor treats empty or oversized LLM responses as a controlled fallback condition and returns deterministic qualification wording while preserving the original scored lead context.

## Concurrency and error handling

Enrichment sources run in separate branches so one unavailable provider does not block all enrichment. Each source returns a structured status:

   - `success`
   - `partial`
   - `empty`
   - `failed`

The aggregation layer carries source warnings into the scored lead and audit record. Scoring continues with available signals and reduces confidence when enrichment is degraded.

Recommended production controls:

   - Add retries and timeouts to real HTTP/API enrichment nodes.
   - Respect enrichment provider rate limits and quota windows.
   - Cache enrichment results where contractually allowed.
   - Route provider failures to an operations or RevOps alert channel.
   - Use workflow-level error handling for unexpected failures.
   - Preserve idempotency keys when creating CRM records, sales tasks or Slack/email notifications.
   - Log source timestamps so sales teams know whether the score used fresh or stale data.

## Security and data handling

- Use n8n Credentials for CRM, enrichment, Slack, email and LLM access.
- Do not hard-code API keys or bearer tokens in workflow JSON.
- Limit outbound enrichment requests to the fields each provider requires.
- Review provider terms for contact enrichment, data retention and regional compliance.
- Avoid sending sensitive CRM notes or private customer data to an LLM.
- Store audit records according to the client's retention and access-control policy.

## Example data

Example files are in `examples/`:

- `examples/sample_input.md`
- `examples/sample_output.md`

The workflow also includes simulation fields for testing degraded paths:

```json
{"company":"partial","tech_stack":"failed"}
```

for `simulate_source_status`, and:

```json
{"crm_upsert":true,"sales_notification":true}
```

for `simulate_action_failures`.
