# n8n Business Flows

This repository contains example n8n workflows for example specific business use case automations.

Each workflow lives in its own folder with:

- A `README.md` describing business value, architecture, and configuration notes.
- An `examples/` directory with sample input/output payloads.

## Current workflows

- `multi-channel-campaign-orchestrator/` – **Multi-Channel Campaign Orchestrator**
    - Takes a single campaign definition (audience, dates, budget, channels) as input.
    - Fans out to set up email, ads, CRM, and internal comms in parallel.
    - Fans back in to build a consolidated summary of what was created where.

Refer to each folder’s `README.md` for workflow-specific details.

## Using the workflows

1. Open your n8n instance (self-hosted or cloud).
2. For a given workflow folder:
- Import the n8n workflow JSON export from that folder (if present), **or** recreate the flow using the folder’s `README.md` and `examples/` as a guide.
3. Configure credentials and environment-specific settings in n8n (e.g. email, Slack, CRM, LLM endpoint).
4. Run the workflow manually or via its configured schedule/trigger.