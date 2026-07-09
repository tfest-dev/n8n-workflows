# Incident Response Orchestrator

This n8n workflow coordinates the first-response process for high-severity incidents across alert intake, ownership routing, enrichment, collaboration, ticketing, status-page handling, on-call notification, operational alerts, and post-incident documentation preparation.

<img src=examples/images/flow-executed-n8n.png alt="Successful execution in n8n"/>

## What the workflow does

The workflow accepts an incident alert, validates the payload, maps the affected service to an owning team and on-call contact, enriches the incident with recent deploy and related-ticket context, then runs the main response actions in parallel. It aggregates the results into a single live incident object that can be written to an incident database, posted into an incident channel, or used by leadership and operations teams during the response.

The repository version uses credential-free placeholder nodes so it can be imported and exercised locally. In a client implementation, those placeholder Code nodes should be replaced with Slack, Microsoft Teams, PagerDuty, Opsgenie, Jira, Linear, Statuspage, incident database, CI/CD, monitoring, and helpdesk connectors as required.

## Business value

- Reduces time-to-coordination during SEV1 and SEV2 incidents.
- Standardises the first-response sequence across alerting, collaboration, ticketing, status updates, and on-call notification.
- Preserves partial-success state so one failed integration does not hide which actions completed.
- Creates a single live incident object with the critical links and owners required during response.
- Captures audit-ready orchestration data for incident dashboards, support review, and post-incident analysis.
- Prepares a post-incident review prompt after the live response object exists, without placing an LLM in the critical response path.

## How the flow works

1. `Manual Trigger (dev)` starts a local demo run.
2. `Incident Webhook (prod)` provides a production-shaped webhook entry point for alerting systems.
3. `Set Example Alert Payload` creates a sample SEV1 alert for local testing.
4. `Validate & Route Incident` checks required fields, normalises severity, parses simulated failure controls, and marks non-SEV1/SEV2 incidents as outside the orchestration policy.
5. `Prepare Incident Control` adds owning-team metadata, on-call routing, retry policy, severity policy, and idempotency keys.
6. The enrichment layer fans out into:
   - `Fetch Recent Deploys (placeholder)`
   - `Fetch Related Tickets (placeholder)`
   - `Resolve On-call Contact (placeholder)`
7. `Build Enriched Incident Object` collects enrichment results and carries source warnings forward.
8. The live orchestration layer fans out into:
   - `Setup Collaboration Channel`
   - `Create or Update Incident Ticket`
   - `Create or Update Status Page`
   - `Notify On-call Engineer`
9. Merge nodes fan the action branches back into one item.
10. `Build Live Incident Object` creates the consolidated incident object with owners, links, failed actions, warnings, and orchestration status.
11. `Prepare Operations Alerts` creates alert payloads for failed action branches or degraded enrichment sources.
12. `Prepare Incident Channel Update` builds the message that would be posted to Slack, Teams, email, or another stakeholder channel.
13. `Prepare Post-Incident Review Prompt` prepares an optional LLM prompt for post-incident documentation after the live response object is available.
14. `Log Audit Snapshot (demo)` creates an audit-ready record for durable storage.

## Client-specific implementation requirements

| Area | Required client decision |
| --- | --- |
| Alert source | Confirm the alerting system, webhook authentication, payload shape, severity naming, and allowed source IPs. |
| Severity policy | Decide which severities trigger orchestration, public status-page updates, executive notification, or customer communication. |
| Ownership mapping | Replace the static service map with service catalogue, on-call rota, incident commander, and escalation-policy data. |
| Collaboration | Choose Slack or Microsoft Teams behaviour: create channel, reuse channel, invite users, pin links, post updates. |
| Ticketing | Choose Jira, Linear, ServiceNow, GitHub Issues, or another incident-record system and define update rules. |
| Status page | Decide public versus internal status-page rules, approval requirements, and customer-communication policy. |
| Incident store | Choose where the live incident object and audit record are retained: database, data warehouse, object storage, Notion, Google Drive, ticketing system, or incident platform. |
| Notifications | Decide which failures go to engineering operations, incident command, SRE, support leadership, or executives. |

## Concurrency and error handling

The enrichment and response actions run in parallel after the incident has been validated and control metadata has been prepared.

Controls included in the workflow:

   - Enrichment branches return structured status instead of blocking the whole response.
   - Collaboration, ticketing, status-page, and on-call branches catch their own failures.
   - `Build Live Incident Object` reports `success`, `partial_success`, or `failed` at orchestration level.
   - Failed branches are carried into `failed_actions` and `operations_alerts`.
   - Retry policy metadata is attached before external actions.
   - Idempotency keys are generated per incident/action to reduce duplicate channel, ticket, status-page, or notification creation after retries.
   - Enrichment warnings are preserved when deploy, related-ticket, or on-call lookup data is unavailable.

Production controls to add when wiring real systems:

   - Configure retry and timeout settings on Slack/Teams, ticketing, status-page, alerting, CI/CD and monitoring API nodes.
   - Retry transient failures such as rate limits, network timeouts and `5xx` responses.
   - Avoid retrying validation failures or unauthorised requests without operator intervention.
   - Route failed branch alerts to an operations channel or incident-command fallback route.
   - Enable workflow-level error handling for unexpected failures outside branch-level controls.
   - Persist request fingerprints and external IDs in the incident store before re-running actions.

## Optional LLM usage

The live incident response path should stay deterministic. An LLM can help after stabilisation by drafting a post-incident review, summarising a timeline, or turning structured incident facts into a report template.

The included `Prepare Post-Incident Review Prompt` node prepares a prompt but does not call a model. In a client implementation, place an LLM step after that node and only send approved incident facts.

Recommended data-handling rules:

   - Do not send raw logs, secrets, customer personal data, credentials, tokens, or full ticket transcripts.
   - Use structured incident facts, sanitised timeline entries, deploy references, and ticket summaries.
   - Keep the incident record, ticketing system, and status-page data as the source of truth.
   - Treat the LLM output as a draft for human review, not as an authoritative postmortem.
   - Use a local or approved private model when incident data is sensitive.

## Example data

Example files are in `examples/`:

- `examples/sample_input.md` — sample SEV1 alert payload.
- `examples/sample_output.md` — sample live incident object.

## Demo notes

To test partial-success behaviour, edit `simulate_failures` in `Set Example Alert Payload`.

Example:

```json
{"status_page": true}
```

The status-page branch will return a failed status, the other branches will continue, and the final live incident object will show `partial_success` with an operations alert for the failed action.
