# n8n Business Workflows

Example n8n workflows for practical business automation use cases. Each workflow is designed as a connector-agnostic starting point that can be adapted for a specific client stack.

The repository is structured as a small automation catalogue. Every workflow includes:

- A workflow-specific `README.md` explaining what it does, why it matters, how it works, implementation requirements, concurrency/error handling, and examples.
- An importable n8n workflow `.json` file.
- An `examples/` folder containing sample inputs and outputs.

## Current workflows

| Workflow | Business value | Import file |
| --- | --- | --- |
| `multi-channel-campaign-orchestrator/` | Converts one approved campaign definition into coordinated setup work across email, ads, CRM, and internal comms. Useful where campaign launches are slow because teams duplicate setup across tools. | `multi-channel-campaign-orchestrator/multi-channel-campaign-orchestrator.json` |
| `reports-weekly-status-local-llm/` | Generates a structured weekly leadership report from operational notes using a local LLM endpoint, with input guardrails, preview/audit checkpoints, failure-alert handling, optional approval placeholder, and email delivery. Useful where teams need consistent reporting without sending sensitive business context to third-party LLM APIs. | `reports-weekly-status-local-llm/reports-weekly-status-local-llm.json` |
| `churn-risk-alerts/` | Builds a customer success churn-risk watchlist from usage, billing, CRM and support signals, with source-status handling, partial-data scoring, optional LLM wording, outreach task preparation and audit logging. Useful where CS teams need proactive retention visibility. | `churn-risk-alerts/churn-risk-alerts.json` |
| `customer-support-digest/` | Produces a daily or weekly support digest from helpdesk tickets, with ticket normalisation, source-status warnings, privacy-aware LLM summarisation, deterministic fallback, leadership notification preparation and audit logging. Useful where leadership and product teams need customer issue visibility without living in the helpdesk. | `customer-support-digest/customer-support-digest.json` |
| `incident-response-orchestrator/` | Coordinates SEV1/SEV2 incident response across alert intake, ownership routing, enrichment, collaboration, ticketing, status-page handling, on-call notification, operational alerts, and post-incident documentation preparation. Useful where incident response needs consistent first-response execution and audit-ready live incident records. | `incident-response-orchestrator/incident-response-orchestrator.json` |
| `invoice-collections-automation/` | Automates overdue-invoice follow-up with customer reminder payloads, account-owner notifications, escalation preparation, finance summaries, idempotency keys and audit records. Useful where finance teams need consistent collections handling without losing visibility of failed actions. | `invoice-collections-automation/invoice-collections-automation.json` |

## Using the workflows

1. Open your n8n instance.
2. Open the folder for the workflow you want to use.
3. Import the workflow JSON file into n8n.
4. Review the workflow-specific `README.md`.
5. Configure credentials and environment-specific settings in n8n, such as email, Slack, CRM, ads, database, helpdesk, or local LLM endpoints.
6. Run the workflow manually first, then enable any schedule, webhook, or production trigger once configuration has been validated.

## Implementation stance

These workflows are portfolio-ready templates for business automation discovery and client adaptation. Real deployments should replace placeholder Code nodes and static data with authenticated connectors, client-specific data sources, proper credentials, durable logging, and production error handling.
