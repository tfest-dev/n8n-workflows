# Multi-Channel Campaign Orchestrator

This workflow coordinates a marketing campaign across multiple channels  (email, ads, CRM, and internal comms) while preserving per-channel status, idempotency controls, and an audit-ready final output.

It is designed as a connector-agnostic n8n template: the included Code nodes simulate the shape of real integrations, while the control layer shows how a client implementation should handle partial success, retries, duplicate prevention, and support visibility.

<img src=examples/images/flow-executed-n8n.png alt="Successful execution in n8n"/>

## Business value

- Reduces manual campaign setup across several disconnected tools.
- Keeps campaign metadata, timing, targeting, budget, and goals consistent across channels.
- Prevents a single failed channel from hiding successful channel work.
- Captures partial success states so operations, marketing, and leadership know exactly what was created and what needs intervention.
- Establishes idempotency and audit patterns that are required before this kind of workflow should touch production marketing systems.

## How this n8n flow works

1. **Manual Trigger (dev)**
   - Starts the workflow for local testing and portfolio demonstration.
   - A production implementation would usually replace or supplement this with a Webhook, form submission, CRM trigger, or campaign approval trigger.

2. **Set Campaign**
   - Provides sample campaign data including campaign ID, audience, dates, budget, enabled channels, goals, and optional simulated failures.
   - In production, this data would come from a campaign intake form, CRM, project-management tool, or campaign planning database.

3. **Validate & Normalise**
   - Confirms required fields are present.
   - Parses numeric and JSON fields.
   - Requires at least one enabled campaign channel.
   - Normalises the payload before orchestration begins.

4. **Prepare Campaign Control**
   - Adds campaign-level control metadata:
     - `campaign_id`
     - per-channel idempotency keys
     - generated timestamp
     - retry policy metadata
   - The idempotency keys are based on campaign/channel/date so a retried workflow can avoid duplicate campaign creation in external systems.

5. **Parallel channel setup**
   - The workflow fans out to four channel branches:
     - `Setup Email Campaign`
     - `Setup Ads`
     - `Setup CRM Campaign`
     - `Notify Internal Comms`
   - Each branch catches its own errors and returns a structured channel result instead of crashing the full workflow.
   - Channel outputs include:
     - `status`: `success`, `failed`, or `skipped`
     - `idempotency_key`
     - retry policy metadata
     - created placeholder entity IDs
     - error message when a branch fails

6. **Fan-in merge**
   - Merge nodes bring the branch outputs back together.
   - The workflow keeps each channel result visible for final reporting.

7. **Build Summary**
   - Builds a Markdown campaign summary.
   - Captures the overall result as:
     - `success`
     - `partial_success`
     - `failed`
   - Lists each channel's status, idempotency key, and any failure reason.

8. **Log Final Output (demo)**
   - Creates an `audit_record` containing:
     - final status
     - channel results
     - idempotency keys
     - retry policy
     - source campaign payload
     - final summary
   - This node is a demo-safe placeholder. In production, replace or extend it with a durable store such as Postgres, Airtable, Google Sheets, Google Drive, Notion, object storage, or a ticketing/support system.

## Client-specific implementation requirements

For a real client deployment, the placeholder Code nodes should be replaced with actual connector or HTTP nodes:

   - Email platform: Mailchimp, HubSpot, Customer.io, Klaviyo, SendGrid, etc.
   - Ads platforms: Meta Ads, Google Ads, LinkedIn Ads, or agency tooling.
   - CRM: HubSpot, Salesforce, Pipedrive, Zoho, etc.
   - Internal comms: Slack, Microsoft Teams, email, or a campaign launch channel.
   - Durable audit store: database, spreadsheet, document store, object storage, or support/ticketing system.

Each external integration should use the generated idempotency key when the destination system supports it. Where the destination does not support native idempotency, store request fingerprints in your audit database and check for existing records before creating new entities.

## Concurrency and error handling

The channel setup branches run in parallel after `Prepare Campaign Control`.

The control layer includes:

   - Per-branch error handling so one failed channel does not hide the status of other branches.
   - Partial success capture in `channel_results` and `summary_markdown`.
   - Retry policy metadata for external APIs with rate limits or transient failures.
   - Per-channel idempotency keys to reduce duplicate campaign creation after retries.
   - Final audit record generation after summary creation.

For production, configure retries on the actual HTTP/native connector nodes. Recommended baseline:

   - Retry on `429`, `408`, network timeout, and `5xx` responses.
   - Do not blindly retry validation errors or `4xx` client errors.
   - Use exponential backoff and a maximum attempt count.
   - Persist every external creation response and request fingerprint to a durable store.
   - Alert on `partial_success` or `failed` so the campaign owner knows which channel needs manual action.

## Example data

Example files are in `examples/`:

- `examples/sample_input.md` — sample campaign intake payload.
- `examples/sample_output.md` — sample final summary and audit shape.

## Demo notes

The included workflow is connector-agnostic. It demonstrates the orchestration and control pattern without requiring live credentials.

To test partial success handling, set `simulate_failures` in `Set Campaign` to something like:

```json
{"ads": true}
```

The ads branch will return a failed status while the other branches continue and the final summary will show `partial_success`.
