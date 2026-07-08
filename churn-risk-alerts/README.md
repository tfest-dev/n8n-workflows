# Churn Risk Alerts

This workflow identifies customer accounts showing churn-risk signals across product usage, billing, CRM and support activity, then prepares a prioritised watchlist for the customer success team.

<img src=examples/images/flow-executed-n8n.png alt="Successful execution in n8n"/>

## Business value

- Gives customer success teams a proactive daily watchlist rather than waiting for cancellation notices.
- Combines usage, renewal, billing, support and sentiment indicators into one account-level view.
- Makes risk scoring transparent, explainable and adjustable for a client’s operating model.
- Helps teams prioritise outreach by risk severity, commercial value and renewal timing.
- Captures source freshness, missing-source warnings and an audit-ready scoring output for later review.

## How the flow works

1. **Manual Trigger (dev)** and **Daily Schedule (prod)** start the workflow for local testing or daily production execution.
2. **Set Example Source Inputs** provides sample usage, billing, CRM and support records. In production, replace these static payloads with authenticated source connectors.
3. **Validate Inputs / Control Config** validates the threshold, top-N limit and lookback window, then adds scoring version, source names, retry policy and data-minimisation guidance.
4. **Collect Usage Signals**, **Collect Billing Signals**, **Collect CRM Signals** and **Collect Support Signals** run as separate source branches. Each branch returns a structured source status of `success`, `partial`, `empty` or `failed`.
5. **Merge Usage + Billing**, **Merge CRM + Support** and **Merge All Sources** bring the source branches back together.
6. **Aggregate Account Signals** combines available records by account ID and carries forward source freshness, missing-source warnings and per-account missing-source lists.
7. **Prepare Source Failure Alerts** prepares operations alert payloads when a source branch is failed, partial or empty.
8. **Score Churn Risk** applies transparent deterministic rules to calculate risk score, severity, reason codes and the account-level idempotency key.
9. **Prepare Optional LLM Outreach Drafts** prepares minimal account-level prompts and demo outreach wording. A client implementation can replace this with a local or approved private LLM call.
10. **Filter / Rank Watchlist** keeps accounts above the configured threshold, sorts by risk score, then MRR, then account ID, and limits the result to the configured top N.
11. **Build Watchlist Summary** creates a Markdown watchlist for customer success review, including source status and operations warnings.
12. **Prepare Outreach Tasks (placeholder)** prepares CRM task payloads with idempotency keys to avoid duplicate follow-up tasks after retries.
13. **Prepare CS Notification (placeholder)** prepares a Slack, Teams or email notification payload with a run-level idempotency key.
14. **Log Audit Snapshot (demo)** builds an audit record containing source statuses, warnings, watchlist output, outreach task payloads and optional LLM policy metadata.

## Client-specific implementation requirements

A client deployment should replace the static source inputs with live integrations:

- Product analytics: Segment, Mixpanel, Amplitude, PostHog, Snowflake, BigQuery or an internal warehouse.
- Billing: Stripe, Xero, QuickBooks, Chargebee, Paddle or a finance database.
- CRM and account ownership: Salesforce, HubSpot, Pipedrive, Zoho or an internal customer table.
- Support and feedback: Zendesk, Intercom, Freshdesk, HubSpot Service Hub, CSAT, NPS or customer health tooling.
- Notification output: Slack, Microsoft Teams, email, CRM tasks or a customer success platform.
- Audit storage: Postgres, warehouse table, Google Sheets, Google Drive, Notion, object storage or a support system.

The scoring weights should be calibrated with the client’s actual churn history. Enterprise accounts may need stronger renewal and sponsor-engagement weighting; self-serve SaaS accounts may need heavier product-usage and payment-signal weighting.

## Concurrency and error handling

The workflow collects usage, billing, CRM and support signals in separate branches so one unavailable source does not block all data collection. Each source branch returns a structured status of `success`, `partial`, `empty` or `failed`, and the aggregation layer includes source freshness and missing-source warnings in the final watchlist.

Production controls represented in the workflow:

   - Source branches are separate and return structured source results.
   - Source status includes `checked_at`, `source_timestamp`, record count, warnings and errors.
   - Scoring continues with available signals when a non-critical source fails.
   - Missing sources are carried into the account record and the final watchlist.
   - Failed, empty or partial sources create operations alert payloads.
   - Outreach tasks and notification payloads include idempotency keys.
   - The final audit record captures source statuses, warnings, scores, reasons, outreach tasks and notification payloads.

Recommended production additions when replacing placeholders with real connector nodes:

   - Add retries and timeouts to HTTP/API source nodes, especially for rate-limited services.
   - Retry transient failures such as `429`, `408`, network timeouts and `5xx` responses.
   - Avoid retrying validation errors or client-side `4xx` responses without review.
   - Store source timestamps from each system so customer success can see whether scores used fresh or stale data.
   - Route source failures to a real operations alert channel such as Slack, Teams, email, PagerDuty or Opsgenie.
   - Configure workflow-level error handling for unexpected failures outside the source branches.
   - Persist idempotency keys and external creation responses before creating CRM tasks or sending notifications.

## Optional LLM usage

The workflow uses deterministic rule-based scoring as the decision source. The optional LLM step is placed after `Score Churn Risk` and is used only for account summary or outreach wording.

The included `Prepare Optional LLM Outreach Drafts` node prepares:

   - A minimal account-level prompt for each scored account.
   - A data policy showing allowed and excluded fields.
   - Demo outreach wording that can be replaced by a local or approved private LLM response.

Use an LLM only after agreeing clear data-handling rules:

   - Send only the fields needed for the explanation or outreach draft.
   - Avoid unnecessary personal data and detailed financial records.
   - Prefer account-level summaries over raw ticket transcripts.
   - Use a local or approved private model where client confidentiality requires it.
   - Keep the rule-based score as the authority, with the LLM used for wording rather than risk decisions.

## Example data

- `examples/sample_input.md`: example source payloads and control configuration.
- `examples/sample_output.md`: example churn-risk watchlist, source warnings, outreach task payload and audit shape.

## Import and test

1. Import `churn-risk-alerts.json` into n8n.
2. Run `Manual Trigger (dev)`.
3. Inspect the output of `Log Audit Snapshot (demo)`.
4. To test partial-source behaviour, set `simulate_source_failures` in `Set Example Source Inputs` to:

```json
{"support": true}
```

The support branch will return a failed source status, scoring will continue with available usage, billing and CRM data, and the final watchlist will include the missing-source warning.
