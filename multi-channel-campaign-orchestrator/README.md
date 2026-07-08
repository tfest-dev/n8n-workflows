# Multi-Channel Campaign Orchestrator

This n8n workflow turns a single campaign definition into coordinated setup activity across email, ads, CRM, and internal communications. It is currently connector-agnostic so can act as a reusable orchestration pattern rather than a fully deployed marketing automation integration.

<img src=examples/images/flow-executed-n8n.png alt="Successful execution in n8n"/>

## What the workflow does

The flow accepts one campaign payload containing the campaign name, audience segment, dates, budget, enabled channels, and goals. It validates and normalises that payload, fans out into parallel channel setup branches, then merges the results into a consolidated campaign summary.

In the repository version, the external integrations are represented by Code nodes with placeholder outputs. That is deliberate: the template demonstrates the business process, data contract, and fan-out/fan-in pattern without requiring specific third-party credentials.

## Business value

Campaign launches often become slow because marketing, sales, CRM, paid media, and internal comms teams each recreate the same campaign context in different tools. This workflow gives a client a controlled launch path:

- One approved campaign definition becomes the source of truth.
- Setup work is executed consistently across channels.
- The final summary gives stakeholders visibility into what was created, skipped, or needs attention.
- The pattern can reduce handoff friction and make campaign launch governance easier to audit.

## How the flow works

1. `Manual Trigger (dev)` starts the workflow for local testing.
2. `Set Campaign` creates an example campaign payload.
3. `Validate & Normalise` checks required fields, parses budget, and converts JSON strings for `channels` and `goals` into objects.
4. Four channel branches run after validation:
   - `Setup Email Campaign`
   - `Setup Ads`
   - `Setup CRM Campaign`
   - `Notify Internal Comms`
5. `Merge Email + Ads`, `Merge Campaign + CRM`, and `Merge Full Campaign` combine the branches back into one enriched item.
6. `Build Summary` creates a Markdown summary of the campaign setup result.

## Client-specific implementation requirements

A production implementation should replace the placeholder Code nodes with real connectors or HTTP calls:

| Area | Example production replacement |
| --- | --- |
| Trigger | Webhook from an approval form, CRM stage change, Airtable/Notion form, or scheduled campaign intake process. |
| Campaign input | CRM, database, spreadsheet, form submission, project-management record, or marketing ops platform. |
| Email branch | Mailchimp, HubSpot, Customer.io, Klaviyo, ActiveCampaign, or custom email API. |
| Ads branch | Google Ads, Meta Ads, LinkedIn Ads, or a media agency intake endpoint. |
| CRM branch | HubSpot, Salesforce, Pipedrive, Attio, or internal CRM API. |
| Internal comms | Slack, Microsoft Teams, email, or ticket creation. |
| Final summary | Slack message, email, database log, campaign record update, or reporting dashboard. |

The client version should also define ownership rules: which team owns campaign approval, who receives failure notifications, and what data is authoritative when systems disagree.

## Concurrency and error handling

The core shape is correct: validate once, fan out, then fan back in. For production, strengthen the control layer:

   - Add per-branch error handling so one failed channel does not hide the status of the other branches.
   - Capture partial success states in the final summary.
   - Add retry policies for external APIs with rate limits or transient failures.
   - Use idempotency keys based on campaign ID to avoid duplicate campaign creation after retries.
   - Log the final output to a durable store for audit and support.

## Example data

Example input and output files live in `examples/`:

- `examples/sample_input.md`
- `examples/sample_output.md`

The workflow currently uses the same shape as the sample input inside the `Set Campaign` node.