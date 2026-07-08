# n8n Business Workflows

Example n8n workflows for practical business automation use cases. Each workflow is designed as a connector-agnostic starting point that can be adapted for a specific client stack.

The repository is structured like a small automation catalogue rather than a random dump of n8n exports. Every workflow includes:

- A workflow-specific `README.md` explaining what it does, why it matters, how it works, implementation requirements, concurrency/error handling, and examples.
- An importable n8n workflow `.json` file.
- An `examples/` folder containing sample inputs and outputs.

## Current workflows

| Workflow | Business value | Import file |
| --- | --- | --- |
| `multi-channel-campaign-orchestrator/` | Converts one approved campaign definition into coordinated setup work across email, ads, CRM, and internal comms. Useful where campaign launches are slow because teams duplicate setup across tools. | `multi-channel-campaign-orchestrator/multi-channel-campaign-orchestrator.json` |
| `reports-weekly-status-local-llm/` | Generates a structured weekly leadership report from operational notes using a local LLM endpoint, with input guardrails, preview/audit checkpoints, failure-alert handling, optional approval placeholder, and email delivery. Useful where teams need consistent reporting without sending sensitive business context to third-party LLM APIs. | `reports-weekly-status-local-llm/reports-weekly-status-local-llm.json` |

## Using the workflows

1. Open your n8n instance.
2. Open the folder for the workflow you want to use.
3. Import the workflow JSON file into n8n.
4. Review the workflow-specific `README.md`.
5. Configure credentials and environment-specific settings in n8n, such as email, Slack, CRM, ads, database, or local LLM endpoints.
6. Run the workflow manually first, then enable any schedule, webhook, or production trigger once configuration has been validated.

## Implementation stance

These workflows are portfolio-ready templates, not plug-and-play SaaS products. The business logic, control flow, and client implementation notes are the value. Real deployments should replace placeholder Code nodes and static data with authenticated connectors, client-specific data sources, proper credentials, and production error handling.
