# Invoice Collections Automation

This workflow automates daily overdue-invoice follow-up across customer reminders, account-owner visibility, escalation preparation, finance summary reporting and audit capture.

<img src=examples/images/flow-executed-n8n.png alt="Successful execution in n8n"/>

## Business value

- Reduces repetitive follow-up work for finance teams.
- Keeps invoice reminders, owner notifications and escalation handling consistent.
- Gives account owners clear visibility into overdue revenue before it becomes a relationship issue.
- Creates a daily collections summary showing overdue totals, ageing buckets, escalations and failed actions.
- Adds idempotency and audit records so retries do not create duplicate reminders or duplicate internal tasks.

## How the workflow works

1. `Manual Trigger (dev)` supports local testing.
2. `Daily Schedule (prod)` runs the workflow daily at 09:00 server time.
3. `Set Example Invoice Inputs` provides sample invoices and configurable thresholds.
4. `Validate Config / Guardrails` parses invoice data, validates thresholds and prepares run-level control settings.
5. `Fetch Overdue Invoices (placeholder)` represents the billing-system source and returns a structured billing-source status.
6. `Prepare Billing Source Alerts` creates operations/finance alert payloads when the billing source is failed, partial or empty.
7. `Normalise & Filter Invoices` removes written-off, invalid, too-small or not-yet-actionable invoices and generates per-invoice idempotency keys.
8. The workflow fans out into parallel action-preparation branches:
   - `Prepare Customer Reminders`
   - `Prepare Account Owner Notifications`
   - `Prepare Internal Logging Rows`
   - `Prepare Escalations`
   - `Prepare Optional LLM Reminder Drafts`
9. Merge nodes combine the branch outputs using compatible merge-by-position configuration.
10. `Build Collections Summary` creates the daily finance summary, bucket totals, escalation list and action-failure list.
11. `Prepare Finance Summary Notification` prepares a Slack/Teams/email-ready payload for the finance channel.
12. `Log Audit Snapshot (demo)` creates an audit-ready record containing source status, eligible invoices, excluded invoices, action results, summary and optional LLM prompt metadata.

## Client-specific implementation requirements

| Area | Required client decision |
| --- | --- |
| Billing source | Replace the placeholder source with Stripe, Xero, QuickBooks, NetSuite, Chargebee or a billing API. |
| Thresholds | Confirm minimum invoice amount, reminder age, escalation age and severe-escalation age. |
| Customer reminders | Configure approved copy, sender identity, delivery provider and suppression rules. |
| Account owner lookup | Decide whether owner data comes from billing, CRM, account records or a fallback routing table. |
| Escalation | Confirm who receives standard and severe escalations, and whether tasks are created in CRM/PM tooling. |
| Logging | Choose durable storage for collections actions, such as a database, spreadsheet, accounting-system note or data warehouse table. |
| Summary delivery | Configure Slack, Teams or email delivery for the finance summary. |
| Retention | Decide how long invoice action history, reminders and audit records are retained. |

## Concurrency and error handling

Customer reminders, account-owner notifications, internal logging, escalation preparation and optional LLM prompt preparation run in parallel after invoices are normalised.

Current controls included in the workflow:

   - Billing-source status is captured as `success`, `partial`, `empty` or `failed`.
   - Source freshness is retained through `checked_at`, `completed_at` and `source_timestamp` fields.
   - Source degradation creates finance/operations alert payloads.
   - Invalid, written-off, below-threshold and below-reminder-age invoices are excluded with reasons.
   - Each action branch catches simulated per-invoice failures and returns structured results.
   - The final summary includes failed actions instead of hiding them.
   - Idempotency keys are generated for customer reminders, owner notifications, internal logging and escalation actions.

Production controls to add when replacing placeholders:

   - Configure retries and timeouts on billing, email, CRM, Slack/Teams and logging nodes.
   - Retry only transient failures such as `429`, `408`, network timeouts and `5xx` responses.
   - Do not blindly retry validation errors, missing customer emails or permission failures.
   - Use idempotency keys where providers support them, and store request fingerprints where they do not.
   - Route failed billing-source runs to workflow-level error handling and a finance operations alert channel.
   - Keep a durable action log so finance can see exactly which reminders, escalations and owner notifications were prepared or sent.

## Optional LLM usage

The workflow prepares optional LLM reminder prompts but does not call an LLM by default.

Recommended usage:

   - Use deterministic templates as the approved baseline for payment-related communications.
   - Use an LLM only for wording/tone suggestions after finance approves the data-handling rules.
   - Send only data-minimised account-level fields.
   - Do not send full payment history, sensitive notes or unnecessary personal data.
   - Keep deterministic workflow rules as the authority for whether an invoice is reminded, escalated or excluded.

## Example data

Example files are in `examples/`:

- `examples/sample_input.md` — sample overdue invoices from a billing system.
- `examples/sample_output.md` — sample daily collections summary and audit shape.

## Demo notes

To test degraded billing-source behaviour, set `simulate_source_status` in `Set Example Invoice Inputs` to `partial`, `empty` or `failed`.

To test per-action failures, set `simulate_failures` to a JSON object such as:

```json
{"customer_reminders":["inv_1002"],"escalations":["inv_1003"]}
```

The workflow should continue, capture the failed action, and include the failure in the final summary and audit record.
